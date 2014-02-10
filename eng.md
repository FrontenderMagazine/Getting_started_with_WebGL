# Getting started with WebGL

[WebGL][1] enables web content to use an API based on [OpenGL ES 2.0][2] to 
perform 3D rendering in an HTML [`canvas`][3] in browsers that support it 
without the use of plug-ins. WebGL programs consist of control code written in 
JavaScript and special effects code(shader code) that is executed on a 
computer's Graphics Processing Unit (GPU). WebGL elements can be mixed with 
other HTML elements and composited with other parts of the page or page 
background.

This article will introduce you to the basics of using WebGL. It's assumed that
you already have an understanding of the mathematics involved in 3D graphics,
and this article doesn't pretend to try to teach you OpenGL itself.

## Preparing to render in 3D

The first thing you need in order to use WebGL to render in 3D is a canvas. The
HTML fragment below establishes a canvas and sets up an onload event handler
that will be used to initialize our WebGL context.

    <body onload="start()">
      <canvas id="glcanvas" width="640" height="480">
        Your browser doesn't appear to support the HTML5 <code>&lt;canvas&gt;</code> element.
      </canvas>
    </body>

### Preparing the WebGL context

The `start()` function, in our JavaScript code, is called after the document is
loaded. Its mission is to set up the WebGL context and start rendering content.

    var gl; // A global variable for the WebGL context

    function start() {
      var canvas = document.getElementById("glcanvas");

      gl = initWebGL(canvas);      // Initialize the GL context
  
      // Only continue if WebGL is available and working
  
      if (gl) {
        gl.clearColor(0.0, 0.0, 0.0, 1.0);                      // Set clear color to black, fully opaque
        gl.enable(gl.DEPTH_TEST);                               // Enable depth testing
        gl.depthFunc(gl.LEQUAL);                                // Near things obscure far things
        gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);      // Clear the color as well as the depth buffer.
      }
    }

The first thing we do here is obtain a reference to the canvas, stashing it in a
variable called `canvas`. Obviously if you don't need to repeatedly reference
the canvas, you should avoid saving this value globally, and just save it in a
local variable or member field of an object.

Once we have the canvas, we call a function called `initWebGL();` this is a
function we'll define momentarily; its job is to initialize the WebGL context.

If the context is successfully initialized, `gl` is a reference to it. In this
case, we set the clear color to black, then clear the context to that color.
After that, the context is configured by setting parameters. In this case, we're
enabling depth testing and specifying that closer objects will obscure objects
that are farther away.

For the purposes of this initial pass at the code, that's all we're going to do.
We'll look at how to actually start doing something a little later.

### Creating a WebGL context

The `initWebGL()` function looks like this:

    function initWebGL(canvas) {
      gl = null;
  
      try {
        // Try to grab the standard context. If it fails, fallback to experimental.
        gl = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");
      }
      catch(e) {}
  
      // If we don't have a GL context, give up now
      if (!gl) {
        alert("Unable to initialize WebGL. Your browser may not support it.");
        gl = null;
      }
  
      return gl;
    }

To obtain a WebGL context for a canvas, we request the context named "webgl"
from the canvas. If this fails, we try the name "experimental-webgl". If that,
too, fails, we display an alert letting the user know they appear not to have
WebGL support. That's all there is to it. At this point, gl is either null
(meaning there is no WebGL context available) or is a reference to the WebGL
context into which we'll be rendering.

> **Note:** The context name "experimental-webgl" is a temporary name for the 
context for use during development of the specification; the name "webgl" will 
be used once the spec is finalized.

At this point, you have enough code that the WebGL context should successfully
initialize, and you should wind up with a big black, empty box, ready and
waiting to receive content.

[Try this example live by clicking here][4] if you're running a WebGL compatible 
browser.

### Resizing the WebGL context

A new WebGL context will set its viewport resolution to the height and width of
its canvas element, without CSS, at the instant the context was obtained.
Editing the style of a canvas element will change its displayed size but will
not change its rendering resolution. Editing the width and height attributes of
a canvas element after the context has been created will also not change the
number of pixels to be drawn. To change the resolution which WebGL renders at,
such as when the user resizes the window of a full-document canvas or you wish
to provide in-app adjustable graphics settings, you will need to call the WebGL
context's `viewport()` function to acknowledge the change.

To modify the rendered resolution of a WebGL context with variables `gl` and
`canvas` as used in the above example:

    gl.viewport(0, 0, canvas.width, canvas.height);

A canvas that is rendered at a different resolution than its CSS style will
experience scaling. Resizing with CSS is mostly useful to save resources by
rendering at a low resolution and allowing the browser to upscale; downscaling
is possible which would produce a super sample antialiasing effect albeit with
naive results and a severe performance cost. It is often best to rely upon the
MSAA and texture filtering implementations of the user's browser, if available
and appropriate.

### See also

* [An introduction to WebGL][5] - Written by Luz Caballero at the DEV.OPERA. 
This article addresses that what WebGL is, how WebGL works and the rendering 
pipeline concept and introduces dome WebGL libraries.
* [An intro to modern OpenGL][6] - A series nice articles about OpenGL written 
by Joe Groff. Joe gives a clear intro about OpenGL from its history to the 
important graphics pipeline concept and provides some examples to demo how 
OpenGL works. If you have no idea about OpenGL, this is a good place to start.

[1]: http://www.khronos.org/webgl/
[2]: http://www.khronos.org/opengles/
[3]: https://developer.mozilla.org/en-US/docs/HTML/Canvas
[4]: https://developer.mozilla.org/samples/webgl/sample1/index.html
[5]: http://dev.opera.com/articles/view/an-introduction-to-webgl/
[6]: http://duriansoftware.com/joe/An-intro-to-modern-OpenGL.-Table-of-Contents.html