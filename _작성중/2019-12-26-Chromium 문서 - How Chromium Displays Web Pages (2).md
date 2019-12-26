---
title:  "Chromium 문서 - How Chromium Displays Web Pages (2)"
excerpt: "How Chromium Displays Web Pages (2) "

categories:
  - Chromium
tags:
  - 크로미움
  - Chromium
last_modified_at: 2019-12-27T17:30:00
---
# The render process

![](https://www.chromium.org/_/rsrc/1342584100941/developers/design-documents/displaying-a-web-page-in-chrome/Renderingintherenderer-v2.png)

Chromium's render process embeds our WebKit port using the glue interface. It does not contain very much code: its job is primarily to be the renderer side of the IPC channel to the browser.

The most important class in the renderer is the RenderView, located in /content/renderer/render_view_impl.cc. This object represents a web page. It handles all navigation-related commands to and from the browser process. It derives from RenderWidget which provides painting and input event handling. The RenderView communicates with the browser process via the global (per render process) RenderProcess object.

### FAQ: What's the difference between RenderWidget and RenderView? 
RenderWidget maps to one WebCore::Widget object by implementing the abstract interface in the glue layer called WebWidgetDelegate.. This is basically a Window on the screen that receives input events and that we paint into. A RenderView inherits from RenderWidget and is the contents of a tab or popup Window. It handles navigational commands in addition to the painting and input events of the widget. There is only one case where a RenderWidget exists without a RenderView, and that's for select boxes on the web page. These are the boxes with the down arrows that pop up a list of options. The select boxes must be rendered using a native window so that they can appear above everything else, and pop out of the frame if necessary. These windows need to paint and receive input, but there isn't a separate "web page" (RenderView) for them.
## Threads in the renderer
Each renderer has two threads (see the multi-process architecture page for a diagram, or threading in Chromium for how to program with them). The render thread is where the main objects such as the RenderView and all WebKit code run. When it communicates to the browser, messages are first sent to the main thread, which in turn dispatches the message to the browser process. Among other things, this allows us to send messages synchronously from the renderer to the browser. This happens for a small set of operations where a result from the browser is required to continue. An example is getting the cookies for a page when requested by JavaScript. The renderer thread will block, and the main thread will queue all messages that are received until the correct response is found. Any messages received in the meantime are subsequently posted to the renderer thread for normal processing.

# The browser process
![](https://www.chromium.org/_/rsrc/1220197831473/developers/design-documents/displaying-a-web-page-in-chrome/rendering%20browser.png)

## Low-level browser process objects
All IPC communication with the render processes is done on the I/O thread of the browser. This thread also handles all network communication which keeps it from interfering with the user interface.

When a RenderProcessHost is initialized on the main thread (where the user interface runs), it creates the new renderer process and a ChannelProxy IPC object with a named pipe to the renderer. This object runs on the I/O thread of the browser, listening to the named pipe to the renderer, and automatically forwards all messages back to the RenderProcessHost on the UI thread. A ResourceMessageFilter will be installed in this channel which will filter out certain messages that can be handled directly on the I/O thread such as network requests. This filtering happens in ResourceMessageFilter::OnMessageReceived.

The RenderProcessHost on the UI thread is responsible for dispatching all view-specific messages to the appropriate RenderViewHost (it handles a limited number of non-view-specific messages itself). This dispatching happens in RenderProcessHost::OnMessageReceived.

## High-level browser process objects
View-specific messages come into RenderViewHost::OnMessageReceived. Most of the messages are handled here, and the rest get forwarded to the RenderWidgetHost base class. These two objects map to the RenderView and the RenderWidget in the renderer (see "The Render Process" above for what these mean). Each platform has a view class (RenderWidgetHostView[Aura|Gtk|Mac|Win]) to implement integration into the native view system.

Above the RenderView/Widget is the WebContents object, and most of the messages actually end up as function calls on that object. A WebContents represents the contents of a webpage. It is the top-level object in the content module, and has the responsibility of displaying a web page in a rectangular view. See the content module pages for more information.

The WebContents object is contained in a TabContentsWrapper. That is in chrome/ and is responsible for a tab.

## Illustrative examples

Additional examples covering navigation and startup are in Getting Around the Chromium Source Code.
### Life of a "set cursor" message
Setting the cursor is an example of a typical message that is sent from the renderer to the browser. In the renderer, here is what happens.
* Set cursor messages are generated by WebKit internally, typically in response to an input event. The set cursor message will start out in RenderWidget::SetCursor in content/renderer/render_widget.cc.
* It will call RenderWidget::Send to dispatch the message. This method is also used by RenderView to send messages to the browser. It will call RenderThread::Send.
* This will call the IPC::SyncChannel which will internally proxy the message to the main thread of the renderer and post it to the named pipe for sending to the browser.
Then the browser takes control:

* The IPC::ChannelProxy in the RenderProcessHost receives all message on the I/O thread of the browser. It first sends them through the ResourceMessageFilter that dispatches network requests and related messages directly on the I/O thread. Since our message is not filtered out, it continues on to the UI thread of the browser (the IPC::ChannelProxy does this internally).
* RenderProcessHost::OnMessageReceived in content/browser/renderer_host/render_process_host_impl.cc gets the messages for all views in the corresponding render process. It handles several types of messages directly, and for the rest forwards to the appropriate RenderViewHost corresponding to the source RenderView that sent the message.
* The message arrives at RenderViewHost::OnMessageReceived in content/browser/renderer_host/render_view_host_impl.cc. Many messages are handled here, but ours is not because it's a message sent from the RenderWidget and handled by the RenderWidgetHost.
* All unhandled messages in RenderViewHost are automatically forwarded to the RenderWidgetHost, including our set cursor message.
* The message map in content/browser/renderer_host/render_view_host_impl.cc finally receives the message in RenderWidgetHost::OnMsgSetCursor and calls the appropriate UI function to set the mouse cursor.

### Life of a "mouse click" message
Sending a mouse click is a typical example of a message going from the browser to the renderer.

* The Windows message is received on the UI thread of the browser by RenderWidgetHostViewWin::OnMouseEvent which then calls ForwardMouseEventToRenderer in the same class.
* The forwarder function packages the input event into a cross-platform WebMouseEvent and ends up sending it to the RenderWidgetHost it is associated with.
* RenderWidgetHost::ForwardInputEvent creates an IPC message ViewMsg_HandleInputEvent, serializes the WebInputEvent to it, and calls RenderWidgetHost::Send.
* This just forwards to the owning RenderProcessHost::Send function, which in turn gives the message to the IPC::ChannelProxy.
* Internally, the IPC::ChannelProxy will proxy the message to the I/O thread of the browser and write it to the named pipe of the corresponding renderer.

Note that many other types of messages are created in the WebContents, especially navigational ones. These follow a similar path from the WebContents to the RenderViewHost.

Then the renderer takes control:

* IPC::Channel on the main thread of the renderer reads the message sent by the browser, and IPC::ChannelProxy proxies to the renderer thread.
* RenderView::OnMessageReceived gets the message. Many types messages are handled here directly. Since the click message is not, it falls through (with all other unhandled messages) to RenderWidget::OnMessageReceived which in turn forwards it to RenderWidget::OnHandleInputEvent.
* The input event is given to WebWidgetImpl::HandleInputEvent where it is converted to a WebKit PlatformMouseEvent class and given to the WebCore::Widget class inside WebKit.
