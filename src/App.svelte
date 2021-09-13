<script lang="ts">
  function copyStylesInline(destinationNode, sourceNode) {
    var containerElements = ["svg", "g"];
    for (var cd = 0; cd < destinationNode.childNodes.length; cd++) {
      var child = destinationNode.childNodes[cd];
      if (containerElements.indexOf(child.tagName) != -1) {
        copyStylesInline(child, sourceNode.childNodes[cd]);
        continue;
      }
      var style =
        sourceNode.childNodes[cd].currentStyle ||
        window.getComputedStyle(sourceNode.childNodes[cd]);
      if (style == "undefined" || style == null) continue;
      for (var st = 0; st < style.length; st++) {
        child.style.setProperty(style[st], style.getPropertyValue(style[st]));
      }
    }
  }

  function triggerDownload(imgURI, fileName) {
    var evt = new MouseEvent("click", {
      view: window,
      bubbles: false,
      cancelable: true,
    });
    var a = document.createElement("a");
    a.setAttribute("download", fileName);
    a.setAttribute("href", imgURI);
    a.setAttribute("target", "_blank");
    a.dispatchEvent(evt);
  }

  function downloadSvg(svg, fileName) {
    var copy = svg.cloneNode(true);
    copyStylesInline(copy, svg);
    var canvas = document.createElement("canvas");
    var bbox = svg.getBBox();
    canvas.width = bbox.width;
    canvas.height = bbox.height;
    var ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, bbox.width, bbox.height);
    var data = new XMLSerializer().serializeToString(copy);
    var DOMURL = window.URL || window.webkitURL;
    var img = new Image();
    var svgBlob = new Blob([data], { type: "image/svg+xml;charset=utf-8" });
    var url = DOMURL.createObjectURL(svgBlob);
    img.onload = function () {
      ctx.drawImage(img, 0, 0);
      DOMURL.revokeObjectURL(url);

      var imgURI = canvas
        .toDataURL("image/png")
        .replace("image/png", "image/octet-stream");
      triggerDownload(imgURI, fileName);
      document.removeChild(canvas);
    };

    img.src = url;
  }
  function download() {
    downloadSvg(svg, `${title}.png`);
  }
  let title = "";
  let svg = null;
</script>

<div class="main">
  <h1>Baptist Hymns Cover Generator</h1>
  <form>
    <input bind:value={title} />
    <button on:click={download}>Save Image</button>
  </form>
  <svg viewBox="0 0 1600 900" bind:this={svg}>
    <rect width="1600" height="900" />
    <text x="800" y="350">{title}</text>
    <text x="800" y="550">Baptist Piano</text>
  </svg>
</div>

<style>
  form {
    display: flex;
    margin: 1em auto;
    width: 500px;
  }
  input {
    padding: 0.5em;
    font-size: 1.5em;
    border: 1px solid black;
    flex-grow: 1;
  }
  button {
    border: 1px solid black;
    border-left: 0px solid black;
    padding: 0.5em;
    font-size: 1.2em;
    background: #4caf50;
    cursor: pointer;
    transition-duration: 0.5s;
  }

  button:hover {
    background: #388e3c;
    color: white;
  }
  .main {
    max-width: 900px;
    width: 90vw;
    margin: 1em auto;
  }

  svg {
    width: 100%;
    display: block;
    /* aspect-ratio: 9 / 16; */
  }

  h1 {
    text-align: center;
    margin: 1em auto;
  }

  text {
    font-size: 6em;
    fill: white;
    text-anchor: middle;
  }

  rect {
    fill: #2196f3;
  }
</style>
