(function(){
  var currentScript = document.currentScript;

  // Multi-instance guard â€” per script tag, not global
  if (currentScript.dataset.hlabsLoaded) return;
  currentScript.dataset.hlabsLoaded = 'true';

  // ---- Sequential script loader ----
  function loadScript(url) {
    return new Promise(function(resolve, reject) {
      var s = document.createElement('script');
      s.src = url;
      s.onload = resolve;
      s.onerror = function() {
        console.warn('[HlabsEmbed] Failed to load: ' + url);
        resolve();
      };
      document.head.appendChild(s);
    });
  }

  function loadScriptsInOrder(urls) {
    var chain = Promise.resolve();
    urls.forEach(function(url) {
      chain = chain.then(function() { return loadScript(url); });
    });
    return chain;
  }

  function runInlineScripts(scripts) {
    // Patch document.write / writeln â€” calling them post-load wipes the whole page.
    // Redirect to insertAdjacentHTML before the currently executing <script>.
    var origWrite = document.write;
    var origWriteln = document.writeln;
    document.write = function() {
      var str = Array.prototype.join.call(arguments, '');
      var cs = document.currentScript;
      if (cs && cs.parentNode) {
        try { cs.insertAdjacentHTML('beforebegin', str); } catch(e) {}
      }
    };
    document.writeln = function() {
      var str = Array.prototype.join.call(arguments, '') + '\n';
      var cs = document.currentScript;
      if (cs && cs.parentNode) {
        try { cs.insertAdjacentHTML('beforebegin', str); } catch(e) {}
      }
    };

    scripts.forEach(function(code) {
      try {
        var s = document.createElement('script');
        s.textContent = code;
        document.body.appendChild(s);
      } catch(e) {
        console.warn('[HlabsEmbed] Inline script error:', e);
      }
    });

    document.write = origWrite;
    document.writeln = origWriteln;
  }

  // ---- Patch addEventListener so DOMContentLoaded/load callbacks fire immediately ----
  function patchReadyListeners() {
    var origDocAdd = document.addEventListener.bind(document);
    var origWinAdd = window.addEventListener.bind(window);

    document.addEventListener = function(type, fn, opts) {
      if (type === 'DOMContentLoaded') {
        setTimeout(fn, 0);
      } else {
        origDocAdd(type, fn, opts);
      }
    };

    window.addEventListener = function(type, fn, opts) {
      if (type === 'load') {
        setTimeout(fn, 0);
      } else {
        origWinAdd(type, fn, opts);
      }
    };

    return function() {
      document.addEventListener = origDocAdd;
      window.addEventListener = origWinAdd;
    };
  }

  // ---- Init ----
  function init() {
    var fontStyle = document.createElement('style');
    fontStyle.setAttribute('data-hlabs-fonts', 'true');
    fontStyle.textContent = `
@font-face {
  font-family: webflow-icons;
  src: url("data:application/x-font-ttf;charset=utf-8;base64,AAEAAAALAIAAAwAwT1MvMg8SBiUAAAC8AAAAYGNtYXDpP+a4AAABHAAAAFxnYXNwAAAAEAAAAXgAAAAIZ2x5ZmhS2XEAAAGAAAADHGhlYWQTFw3HAAAEnAAAADZoaGVhCXYFgQAABNQAAAAkaG10eCe4A1oAAAT4AAAAMGxvY2EDtALGAAAFKAAAABptYXhwABAAPgAABUQAAAAgbmFtZSoCsMsAAAVkAAABznBvc3QAAwAAAAAHNAAAACAAAwP4AZAABQAAApkCzAAAAI8CmQLMAAAB6wAzAQkAAAAAAAAAAAAAAAAAAAABEAAAAAAAAAAAAAAAAAAAAABAAADpAwPA/8AAQAPAAEAAAAABAAAAAAAAAAAAAAAgAAAAAAADAAAAAwAAABwAAQADAAAAHAADAAEAAAAcAAQAQAAAAAwACAACAAQAAQAg5gPpA//9//8AAAAAACDmAOkA//3//wAB/+MaBBcIAAMAAQAAAAAAAAAAAAAAAAABAAH//wAPAAEAAAAAAAAAAAACAAA3OQEAAAAAAQAAAAAAAAAAAAIAADc5AQAAAAABAAAAAAAAAAAAAgAANzkBAAAAAAEBIAAAAyADgAAFAAAJAQcJARcDIP5AQAGA/oBAAcABwED+gP6AQAABAOAAAALgA4AABQAAEwEXCQEH4AHAQP6AAYBAAcABwED+gP6AQAAAAwDAAOADQALAAA8AHwAvAAABISIGHQEUFjMhMjY9ATQmByEiBh0BFBYzITI2PQE0JgchIgYdARQWMyEyNj0BNCYDIP3ADRMTDQJADRMTDf3ADRMTDQJADRMTDf3ADRMTDQJADRMTAsATDSANExMNIA0TwBMNIA0TEw0gDRPAEw0gDRMTDSANEwAAAAABAJ0AtAOBApUABQAACQIHCQEDJP7r/upcAXEBcgKU/usBFVz+fAGEAAAAAAL//f+9BAMDwwAEAAkAABcBJwEXAwE3AQdpA5ps/GZsbAOabPxmbEMDmmz8ZmwDmvxmbAOabAAAAgAA/8AEAAPAAB0AOwAABSInLgEnJjU0Nz4BNzYzMTIXHgEXFhUUBw4BBwYjNTI3PgE3NjU0Jy4BJyYjMSIHDgEHBhUUFx4BFxYzAgBqXV6LKCgoKIteXWpqXV6LKCgoKIteXWpVSktvICEhIG9LSlVVSktvICEhIG9LSlVAKCiLXl1qal1eiygoKCiLXl1qal1eiygoZiEgb0tKVVVKS28gISEgb0tKVVVKS28gIQABAAABwAIAA8AAEgAAEzQ3PgE3NjMxFSIHDgEHBhUxIwAoKIteXWpVSktvICFmAcBqXV6LKChmISBvS0pVAAAAAgAA/8AFtgPAADIAOgAAARYXHgEXFhUUBw4BBwYHIxUhIicuAScmNTQ3PgE3NjMxOAExNDc+ATc2MzIXHgEXFhcVATMJATMVMzUEjD83NlAXFxYXTjU1PQL8kz01Nk8XFxcXTzY1PSIjd1BQWlJJSXInJw3+mdv+2/7c25MCUQYcHFg5OUA/ODlXHBwIAhcXTzY1PTw1Nk8XF1tQUHcjIhwcYUNDTgL+3QFt/pOTkwABAAAAAQAAmM7nP18PPPUACwQAAAAAANciZKUAAAAA1yJkpf/9/70FtgPDAAAACAACAAAAAAAAAAEAAAPA/8AAAAW3//3//QW2AAEAAAAAAAAAAAAAAAAAAAAMBAAAAAAAAAAAAAAAAgAAAAQAASAEAADgBAAAwAQAAJ0EAP/9BAAAAAQAAAAFtwAAAAAAAAAKABQAHgAyAEYAjACiAL4BFgE2AY4AAAABAAAADAA8AAMAAAAAAAIAAAAAAAAAAAAAAAAAAAAAAAAADgCuAAEAAAAAAAEADQAAAAEAAAAAAAIABwCWAAEAAAAAAAMADQBIAAEAAAAAAAQADQCrAAEAAAAAAAUACwAnAAEAAAAAAAYADQBvAAEAAAAAAAoAGgDSAAMAAQQJAAEAGgANAAMAAQQJAAIADgCdAAMAAQQJAAMAGgBVAAMAAQQJAAQAGgC4AAMAAQQJAAUAFgAyAAMAAQQJAAYAGgB8AAMAAQQJAAoANADsd2ViZmxvdy1pY29ucwB3AGUAYgBmAGwAbwB3AC0AaQBjAG8AbgBzVmVyc2lvbiAxLjAAVgBlAHIAcwBpAG8AbgAgADEALgAwd2ViZmxvdy1pY29ucwB3AGUAYgBmAGwAbwB3AC0AaQBjAG8AbgBzd2ViZmxvdy1pY29ucwB3AGUAYgBmAGwAbwB3AC0AaQBjAG8AbgBzUmVndWxhcgBSAGUAZwB1AGwAYQByd2ViZmxvdy1pY29ucwB3AGUAYgBmAGwAbwB3AC0AaQBjAG8AbgBzRm9udCBnZW5lcmF0ZWQgYnkgSWNvTW9vbi4ARgBvAG4AdAAgAGcAZQBuAGUAcgBhAHQAZQBkACAAYgB5ACAASQBjAG8ATQBvAG8AbgAuAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==") format("truetype");
  font-weight: normal;
  font-style: normal;
}
@font-face {
  font-family: Sohne;
  src: url("https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ac23d3167f9fa9cab171f_So%CC%88hne-Buch.otf") format("opentype");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: Sohne;
  src: url("https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ac448cb1547e97cf00eca_So%CC%88hne-Halbfett.otf") format("opentype");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: Sohne;
  src: url("https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ac4621b9dbe921414f8d3_So%CC%88hne-Dreiviertelfett.otf") format("opentype");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: Sohne;
  src: url("https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ac47d0909131f19464b05_So%CC%88hne-Extrafett.otf") format("opentype");
  font-weight: 800;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: Sohne;
  src: url("https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ac49789e3aebefe5e5ae8_So%CC%88hne-Fett.otf") format("opentype");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
`;
    document.head.appendChild(fontStyle);

    var layoutStyle = document.createElement('style');
    layoutStyle.setAttribute('data-hlabs-styles', 'true');
    layoutStyle.textContent = `
  /* ==========================================================================
     1. BASE & TYPOGRAPHY STYLES
     ========================================================================== */
  .hs_main-wrapper {
    color: #172349;
  }

  .hs_main-wrapper a,
  .hs_main-wrapper a:link,
  .hs_main-wrapper a:visited {
    text-decoration: inherit;
  }

  .hs_main-wrapper li::marker {
    color: #172349;
  }

  .hs_main-wrapper p {
    margin-bottom: 0px;
  }

  /* ==========================================================================
     2. UI COMPONENTS & INTERACTION
     ========================================================================== */
  /* Tabs */
  .hs_quiz_persona_tab-link:nth-of-type(1):hover {
    background-color: hsl(78deg 60% 67% / 40%);
  }

  .hs_quiz_persona_tab-link:nth-of-type(1).w--current,
  .hs_quiz_persona_tab-link:nth-of-type(1):active {
    background-color: #c0de7a;
  }

  .hs_quiz_persona_tab-link:nth-of-type(2):hover {
    background-color: hsl(200deg 80% 75% / 40%);
  }

  .hs_quiz_persona_tab-link:nth-of-type(2).w--current,
  .hs_quiz_persona_tab-link:nth-of-type(2):active {
    background-color: #8bcff2;
  }

  /* Buttons & Backgrounds */
  [data-hs-action='start-playing'] {
    transition:
      background-color 0.4s cubic-bezier(0.165, 0.84, 0.44, 1),
      color 0.4s cubic-bezier(0.165, 0.84, 0.44, 1);
  }

  [data-hs-action='start-playing']:hover {
    background-color: #172349;
    color: #ffffff;
  }

  [data-hs-bg] {
    display: none;
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  /* ==========================================================================
     3. QUIZ LOGIC & STATE VISIBILITY
     ========================================================================== */
  /* 1. Hide Persona, Assessment, and Outcome wrappers on load */
  [data-hs-view='persona'],
  [data-hs-view='assessment'],
  [data-hs-view='outcome'] {
    display: none;
  }

  /* 2. Hide all category wrappers on load */
  [data-hs-category] {
    display: none;
  }

  /* 3. Hide all Question Sets by default */
  .hs_quiz_assessment_set {
    display: none;
  }

  /* 4. Show a Question Set when active */
  .hs_quiz_assessment_set.is-active {
    display: flex;
  }

  /* 5. Hide the feedback block by default */
  .hs_quiz_assessment_set [data-hs-block='feedback'] {
    display: none;
  }

  /* 6. Hide specific messages by default */
  [data-hs-feedback='correct'],
  [data-hs-feedback='incorrect'] {
    display: none;
  }

  /* 7. Hide all individual outcome cards by default */
  [data-hs-outcome] {
    display: none;
  }

  /* ==========================================================================
     4. CARD & ACCORDION ELEMENTS
     ========================================================================== */
  span.hs_quiz_assessment_qa-label {
    margin-right: 4px;
  }

  .hs_quiz_outcome_accordion.is-active .hs_quiz_outcome_accordion_content {
    grid-template-rows: 1fr;
  }

  .hs_quiz_outcome_accordion.is-active .hs_quiz_outcome_accordion_line.is-2 {
    width: 0%;
  }

  /* ==========================================================================
     5. CATEGORY THEMES
     ========================================================================== */
  /* Agriculture */
  [data-hs-category='agriculture'] .hs_quiz_assessment_progress-fill {
    background-color: #81c24e;
  }

  [data-hs-category='agriculture'] .hs_quiz_assessment_q-radio:hover {
    border-color: #c0de7a;
    background-color: hsl(78deg 60% 67% / 20%);
  }

  [data-hs-category='agriculture'] .hs_quiz_assessment_q-radio:hover .hs_quiz_assessment_q-radio-btn {
    border-color: #c0de7a;
    background-color: #c0de7a;
  }

  /* Healthcare */
  [data-hs-category='healthcare'] .hs_quiz_assessment_progress-fill {
    background-color: #8bcff2;
  }

  [data-hs-category='healthcare'] .hs_quiz_assessment_q-radio:hover {
    border-color: #8bcff2;
    background-color: hsl(200deg 80% 75% / 20%);
  }

  [data-hs-category='healthcare'] .hs_quiz_assessment_q-radio:hover .hs_quiz_assessment_q-radio-btn {
    border-color: #8bcff2;
    background-color: #8bcff2;
  }

  /* ==========================================================================
     6. RESPONSIVE / MEDIA QUERIES
     ========================================================================== */
  /* 1. MOBILE ONLY (Heights below 670px) */
  @media screen and (max-width: 767px) and (max-height: 670px) {
    .hs_quiz_assessment_text.is-q {
      flex-shrink: 0 !important;
      font-size: 13px !important;
    }

    .hs_quiz_assessment_q-radio-label {
      font-size: 13px !important;
    }

    .hs_quiz_persona_tab-info {
      max-height: 150px !important;
      overflow: auto !important;
    }

    /*  .hs_quiz_intro_content-wrap {
      max-height: 220px !important;
      overflow: auto !important;
    }*/

    .hs_quiz_assessment_q-main {
      max-height: 242px !important;
      overflow: auto !important;
    }
    .hs_quiz_outcome_accordion {
      max-height: 228px;
    }
  }

  /* 2. MOBILE ONLY (Heights above 667px) */
  @media screen and (max-width: 767px) and (min-height: 670px) {
    .hs_quiz_assessment_text.is-q {
      /* max-height: 80px; */
      overflow: auto !important;
    }

    .hs_quiz_persona_tab-info {
      max-height: 180px !important;
      overflow: auto !important;
    }

    .hs_quiz_intro_content-wrap {
      max-height: 320px !important;
      overflow: auto !important;
    }

    .hs_quiz_assessment_q-main {
      max-height: none !important;
      overflow: auto !important;
    }
  }

  /* 2. TABLET ONLY (Width: 768px - 991px | Heights 600px up to 765px) */
  @media screen and (min-width: 768px) and (max-width: 991px) and (min-height: 600px) and (max-height: 765px) {
    .hs_quiz_persona_tab-info {
      max-height: 180px !important;
      overflow: auto !important;
    }

    .hs_quiz_assessment_text.is-q {
      flex-shrink: 0 !important;
    }

    .hs_quiz_assessment_text {
      font-size: 14px !important;
    }

    .hs_quiz_assessment_q-main {
      max-height: 210px !important;
      overflow: auto !important;
    }

    /*.hs_quiz_outcome_info {
      overflow: auto !important;
      max-height: 140px !important;
    }*/
    .hs_quiz_outcome_accordion {
      max-height: 188px;
    }
  }

  /* 3. DESKTOP ONLY (Width: 992px+ | Heights below 770px) */
  @media screen and (min-width: 992px) and (max-height: 770px) {
    .hs_quiz_assessment_text.is-q {
      flex-shrink: 0 !important;
    }

    .hs_quiz_persona_tab-info {
      max-height: 140px !important;
      overflow: auto !important;
    }

    .hs_quiz_assessment_text {
      font-size: 14px !important;
    }

    .hs_quiz_assessment_card {
      margin-top: 40px !important;
    }

    .hs_quiz_intro_content-wrap {
      max-height: 150px !important;
      overflow: auto !important;
    }

    .hs_quiz_assessment_q-main {
      max-height: 210px !important;
      overflow: auto !important;
    }

    /*.hs_quiz_outcome_info {
      overflow: auto !important;
      max-height: 140px !important;
    }*/
    .hs_quiz_outcome_accordion {
      max-height: 188px;
    }
  }

  .hs_quiz_outcome_info-text a {
    text-decoration: underline !important;
    font-style: normal;
  }
  .hs_quiz_outcome_info-text em {
    font-style: normal;
  }
  .article > * {
    margin-bottom: 0px;
  }

html {
  -webkit-text-size-adjust: 100%;
  -ms-text-size-adjust: 100%;
  font-family: sans-serif;
}

body {
  margin: 0;
}

article, aside, details, figcaption, figure, footer, header, hgroup, main, menu, nav, section, summary {
  display: block;
}

audio, canvas, progress, video {
  vertical-align: baseline;
  display: inline-block;
}

audio:not([controls]) {
  height: 0;
  display: none;
}

[hidden], template {
  display: none;
}

a {
  background-color: #0000;
}

a:active, a:hover {
  outline: 0;
}

abbr[title] {
  border-bottom: 1px dotted;
}

b, strong {
  font-weight: bold;
}

dfn {
  font-style: italic;
}

h1 {
  margin: .67em 0;
  font-size: 2em;
}

mark {
  color: #000;
  background: #ff0;
}

small {
  font-size: 80%;
}

sub, sup {
  vertical-align: baseline;
  font-size: 75%;
  line-height: 0;
  position: relative;
}

sup {
  top: -.5em;
}

sub {
  bottom: -.25em;
}

img {
  border: 0;
}

svg:not(:root) {
  overflow: hidden;
}

hr {
  box-sizing: content-box;
  height: 0;
}

pre {
  overflow: auto;
}

code, kbd, pre, samp {
  font-family: monospace;
  font-size: 1em;
}

button, input, optgroup, select, textarea {
  color: inherit;
  font: inherit;
  margin: 0;
}

button {
  overflow: visible;
}

button, select {
  text-transform: none;
}

button, html input[type="button"], input[type="reset"] {
  -webkit-appearance: button;
  cursor: pointer;
}

button[disabled], html input[disabled] {
  cursor: default;
}

button::-moz-focus-inner, input::-moz-focus-inner {
  border: 0;
  padding: 0;
}

input {
  line-height: normal;
}

input[type="checkbox"], input[type="radio"] {
  box-sizing: border-box;
  padding: 0;
}

input[type="number"]::-webkit-inner-spin-button, input[type="number"]::-webkit-outer-spin-button {
  height: auto;
}

input[type="search"] {
  -webkit-appearance: none;
}

input[type="search"]::-webkit-search-cancel-button, input[type="search"]::-webkit-search-decoration {
  -webkit-appearance: none;
}

legend {
  border: 0;
  padding: 0;
}

textarea {
  overflow: auto;
}

optgroup {
  font-weight: bold;
}

table {
  border-collapse: collapse;
  border-spacing: 0;
}

td, th {
  padding: 0;
}



[class^="w-icon-"], [class*=" w-icon-"] {
  speak: none;
  font-variant: normal;
  text-transform: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-style: normal;
  font-weight: normal;
  line-height: 1;
  font-family: webflow-icons !important;
}

.w-icon-slider-right:before {
  content: "î˜€";
}

.w-icon-slider-left:before {
  content: "î˜";
}

.w-icon-nav-menu:before {
  content: "î˜‚";
}

.w-icon-arrow-down:before, .w-icon-dropdown-toggle:before {
  content: "î˜ƒ";
}

.w-icon-file-upload-remove:before {
  content: "î¤€";
}

.w-icon-file-upload-icon:before {
  content: "î¤ƒ";
}

* {
  box-sizing: border-box;
}

html {
  height: 100%;
}

body {
  color: #333;
  background-color: #fff;
  min-height: 100%;
  margin: 0;
  font-family: Arial, sans-serif;
  font-size: 14px;
  line-height: 20px;
}

img {
  vertical-align: middle;
  max-width: 100%;
  display: inline-block;
}

html.w-mod-touch * {
  background-attachment: scroll !important;
}

.w-block {
  display: block;
}

.w-inline-block {
  max-width: 100%;
  display: inline-block;
}

.w-clearfix:before, .w-clearfix:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-clearfix:after {
  clear: both;
}

.w-hidden {
  display: none;
}

.w-button {
  color: #fff;
  line-height: inherit;
  cursor: pointer;
  background-color: #3898ec;
  border: 0;
  border-radius: 0;
  padding: 9px 15px;
  text-decoration: none;
  display: inline-block;
}

input.w-button {
  -webkit-appearance: button;
}

:where(button.w-popover-trigger) {
  color: inherit;
  cursor: pointer;
  font: inherit;
  line-height: inherit;
  text-align: inherit;
  -webkit-text-decoration: inherit;
  text-decoration: inherit;
  background-color: #0000;
  border: 0;
  margin: 0;
  padding: 0;
}

.w-popover {
  position-anchor: none;
}

.w-popover[popover]:not(:popover-open) {
  display: none;
}

html[data-w-dynpage] [data-w-cloak] {
  color: #0000 !important;
}

.w-code-block {
  margin: unset;
}

pre.w-code-block code {
  all: inherit;
}

pre.w-code-block code > span {
  display: block !important;
}

.w-optimization {
  display: contents;
}

.w-webflow-badge, .w-webflow-badge > img {
  box-sizing: unset;
  width: unset;
  height: unset;
  max-height: unset;
  max-width: unset;
  min-height: unset;
  min-width: unset;
  margin: unset;
  padding: unset;
  float: unset;
  clear: unset;
  border: unset;
  border-radius: unset;
  background: unset;
  background-image: unset;
  background-position: unset;
  background-size: unset;
  background-repeat: unset;
  background-origin: unset;
  background-clip: unset;
  background-attachment: unset;
  background-color: unset;
  box-shadow: unset;
  transform: unset;
  direction: unset;
  font-family: unset;
  font-weight: unset;
  color: unset;
  font-size: unset;
  line-height: unset;
  font-style: unset;
  font-variant: unset;
  text-align: unset;
  letter-spacing: unset;
  -webkit-text-decoration: unset;
  text-decoration: unset;
  text-indent: unset;
  text-transform: unset;
  list-style-type: unset;
  text-shadow: unset;
  vertical-align: unset;
  cursor: unset;
  white-space: unset;
  word-break: unset;
  word-spacing: unset;
  word-wrap: unset;
  transition: unset;
}

.w-webflow-badge {
  white-space: nowrap;
  cursor: pointer;
  box-shadow: 0 0 0 1px #0000001a, 0 1px 3px #0000001a;
  visibility: visible !important;
  opacity: 1 !important;
  z-index: 2147483647 !important;
  color: #aaadb0 !important;
  width: auto !important;
  height: auto !important;
  overflow: unset !important;
  background-color: #fff !important;
  border-radius: 3px !important;
  margin: 0 !important;
  padding: 6px !important;
  font-size: 12px !important;
  line-height: 14px !important;
  text-decoration: none !important;
  display: inline-block !important;
  position: fixed !important;
  inset: auto 12px 12px auto !important;
  transform: none !important;
}

.w-webflow-badge > img {
  position: unset;
  visibility: unset !important;
  opacity: 1 !important;
  vertical-align: middle !important;
  display: inline-block !important;
}

h1, h2, h3, h4, h5, h6 {
  margin-bottom: 10px;
  font-weight: bold;
}

h1 {
  margin-top: 20px;
  font-size: 38px;
  line-height: 44px;
}

h2 {
  margin-top: 20px;
  font-size: 32px;
  line-height: 36px;
}

h3 {
  margin-top: 20px;
  font-size: 24px;
  line-height: 30px;
}

h4 {
  margin-top: 10px;
  font-size: 18px;
  line-height: 24px;
}

h5 {
  margin-top: 10px;
  font-size: 14px;
  line-height: 20px;
}

h6 {
  margin-top: 10px;
  font-size: 12px;
  line-height: 18px;
}

p {
  margin-top: 0;
  margin-bottom: 10px;
}

blockquote {
  border-left: 5px solid #e2e2e2;
  margin: 0 0 10px;
  padding: 10px 20px;
  font-size: 18px;
  line-height: 22px;
}

figure {
  margin: 0 0 10px;
}

figcaption {
  text-align: center;
  margin-top: 5px;
}

ul, ol {
  margin-top: 0;
  margin-bottom: 10px;
  padding-left: 40px;
}

.w-list-unstyled {
  padding-left: 0;
  list-style: none;
}

.w-embed:before, .w-embed:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-embed:after {
  clear: both;
}

.w-video {
  width: 100%;
  padding: 0;
  position: relative;
}

.w-video iframe, .w-video object, .w-video embed {
  border: none;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

fieldset {
  border: 0;
  margin: 0;
  padding: 0;
}

button, [type="button"], [type="reset"] {
  cursor: pointer;
  -webkit-appearance: button;
  border: 0;
}

.w-form {
  margin: 0 0 15px;
}

.w-form-done {
  text-align: center;
  background-color: #ddd;
  padding: 20px;
  display: none;
}

.w-form-fail {
  background-color: #ffdede;
  margin-top: 10px;
  padding: 10px;
  display: none;
}

label {
  margin-bottom: 5px;
  font-weight: bold;
  display: block;
}

.w-input, .w-select {
  color: #333;
  vertical-align: middle;
  background-color: #fff;
  border: 1px solid #ccc;
  width: 100%;
  height: 38px;
  margin-bottom: 10px;
  padding: 8px 12px;
  font-size: 14px;
  line-height: 1.42857;
  display: block;
}

.w-input::placeholder, .w-select::placeholder {
  color: #999;
}

.w-input:focus, .w-select:focus {
  border-color: #3898ec;
  outline: 0;
}

.w-input[disabled], .w-select[disabled], .w-input[readonly], .w-select[readonly], fieldset[disabled] .w-input, fieldset[disabled] .w-select {
  cursor: not-allowed;
}

.w-input[disabled]:not(.w-input-disabled), .w-select[disabled]:not(.w-input-disabled), .w-input[readonly], .w-select[readonly], fieldset[disabled]:not(.w-input-disabled) .w-input, fieldset[disabled]:not(.w-input-disabled) .w-select {
  background-color: #eee;
}

textarea.w-input, textarea.w-select {
  height: auto;
}

.w-select {
  background-color: #f3f3f3;
}

.w-select[multiple] {
  height: auto;
}

.w-form-label {
  cursor: pointer;
  margin-bottom: 0;
  font-weight: normal;
  display: inline-block;
}

.w-radio {
  margin-bottom: 5px;
  padding-left: 20px;
  display: block;
}

.w-radio:before, .w-radio:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-radio:after {
  clear: both;
}

.w-radio-input {
  float: left;
  margin: 3px 0 0 -20px;
  line-height: normal;
}

.w-file-upload {
  margin-bottom: 10px;
  display: block;
}

.w-file-upload-input {
  opacity: 0;
  z-index: -100;
  width: .1px;
  height: .1px;
  position: absolute;
  overflow: hidden;
}

.w-file-upload-default, .w-file-upload-uploading, .w-file-upload-success {
  color: #333;
  display: inline-block;
}

.w-file-upload-error {
  margin-top: 10px;
  display: block;
}

.w-file-upload-default.w-hidden, .w-file-upload-uploading.w-hidden, .w-file-upload-error.w-hidden, .w-file-upload-success.w-hidden {
  display: none;
}

.w-file-upload-uploading-btn {
  cursor: pointer;
  background-color: #fafafa;
  border: 1px solid #ccc;
  margin: 0;
  padding: 8px 12px;
  font-size: 14px;
  font-weight: normal;
  display: flex;
}

.w-file-upload-file {
  background-color: #fafafa;
  border: 1px solid #ccc;
  flex-grow: 1;
  justify-content: space-between;
  margin: 0;
  padding: 8px 9px 8px 11px;
  display: flex;
}

.w-file-upload-file-name {
  font-size: 14px;
  font-weight: normal;
  display: block;
}

.w-file-remove-link {
  cursor: pointer;
  width: auto;
  height: auto;
  margin-top: 3px;
  margin-left: 10px;
  padding: 3px;
  display: block;
}

.w-icon-file-upload-remove {
  margin: auto;
  font-size: 10px;
}

.w-file-upload-error-msg {
  color: #ea384c;
  padding: 2px 0;
  display: inline-block;
}

.w-file-upload-info {
  padding: 0 12px;
  line-height: 38px;
  display: inline-block;
}

.w-file-upload-label {
  cursor: pointer;
  background-color: #fafafa;
  border: 1px solid #ccc;
  margin: 0;
  padding: 8px 12px;
  font-size: 14px;
  font-weight: normal;
  display: inline-block;
}

.w-icon-file-upload-icon, .w-icon-file-upload-uploading {
  width: 20px;
  margin-right: 8px;
  display: inline-block;
}

.w-icon-file-upload-uploading {
  height: 20px;
}

.w-container {
  max-width: 940px;
  margin-left: auto;
  margin-right: auto;
}

.w-container:before, .w-container:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-container:after {
  clear: both;
}

.w-container .w-row {
  margin-left: -10px;
  margin-right: -10px;
}

.w-row:before, .w-row:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-row:after {
  clear: both;
}

.w-row .w-row {
  margin-left: 0;
  margin-right: 0;
}

.w-col {
  float: left;
  width: 100%;
  min-height: 1px;
  padding-left: 10px;
  padding-right: 10px;
  position: relative;
}

.w-col .w-col {
  padding-left: 0;
  padding-right: 0;
}

.w-col-1 {
  width: 8.33333%;
}

.w-col-2 {
  width: 16.6667%;
}

.w-col-3 {
  width: 25%;
}

.w-col-4 {
  width: 33.3333%;
}

.w-col-5 {
  width: 41.6667%;
}

.w-col-6 {
  width: 50%;
}

.w-col-7 {
  width: 58.3333%;
}

.w-col-8 {
  width: 66.6667%;
}

.w-col-9 {
  width: 75%;
}

.w-col-10 {
  width: 83.3333%;
}

.w-col-11 {
  width: 91.6667%;
}

.w-col-12 {
  width: 100%;
}

.w-hidden-main {
  display: none !important;
}

@media screen and (max-width: 991px) {
  .w-container {
    max-width: 728px;
  }

  .w-hidden-main {
    display: inherit !important;
  }

  .w-hidden-medium {
    display: none !important;
  }

  .w-col-medium-1 {
    width: 8.33333%;
  }

  .w-col-medium-2 {
    width: 16.6667%;
  }

  .w-col-medium-3 {
    width: 25%;
  }

  .w-col-medium-4 {
    width: 33.3333%;
  }

  .w-col-medium-5 {
    width: 41.6667%;
  }

  .w-col-medium-6 {
    width: 50%;
  }

  .w-col-medium-7 {
    width: 58.3333%;
  }

  .w-col-medium-8 {
    width: 66.6667%;
  }

  .w-col-medium-9 {
    width: 75%;
  }

  .w-col-medium-10 {
    width: 83.3333%;
  }

  .w-col-medium-11 {
    width: 91.6667%;
  }

  .w-col-medium-12 {
    width: 100%;
  }

  .w-col-stack {
    width: 100%;
    left: auto;
    right: auto;
  }
}

@media screen and (max-width: 767px) {
  .w-hidden-main, .w-hidden-medium {
    display: inherit !important;
  }

  .w-hidden-small {
    display: none !important;
  }

  .w-row, .w-container .w-row {
    margin-left: 0;
    margin-right: 0;
  }

  .w-col {
    width: 100%;
    left: auto;
    right: auto;
  }

  .w-col-small-1 {
    width: 8.33333%;
  }

  .w-col-small-2 {
    width: 16.6667%;
  }

  .w-col-small-3 {
    width: 25%;
  }

  .w-col-small-4 {
    width: 33.3333%;
  }

  .w-col-small-5 {
    width: 41.6667%;
  }

  .w-col-small-6 {
    width: 50%;
  }

  .w-col-small-7 {
    width: 58.3333%;
  }

  .w-col-small-8 {
    width: 66.6667%;
  }

  .w-col-small-9 {
    width: 75%;
  }

  .w-col-small-10 {
    width: 83.3333%;
  }

  .w-col-small-11 {
    width: 91.6667%;
  }

  .w-col-small-12 {
    width: 100%;
  }
}

@media screen and (max-width: 479px) {
  .w-container {
    max-width: none;
  }

  .w-hidden-main, .w-hidden-medium, .w-hidden-small {
    display: inherit !important;
  }

  .w-hidden-tiny {
    display: none !important;
  }

  .w-col {
    width: 100%;
  }

  .w-col-tiny-1 {
    width: 8.33333%;
  }

  .w-col-tiny-2 {
    width: 16.6667%;
  }

  .w-col-tiny-3 {
    width: 25%;
  }

  .w-col-tiny-4 {
    width: 33.3333%;
  }

  .w-col-tiny-5 {
    width: 41.6667%;
  }

  .w-col-tiny-6 {
    width: 50%;
  }

  .w-col-tiny-7 {
    width: 58.3333%;
  }

  .w-col-tiny-8 {
    width: 66.6667%;
  }

  .w-col-tiny-9 {
    width: 75%;
  }

  .w-col-tiny-10 {
    width: 83.3333%;
  }

  .w-col-tiny-11 {
    width: 91.6667%;
  }

  .w-col-tiny-12 {
    width: 100%;
  }
}

.w-widget {
  position: relative;
}

.w-widget-map {
  width: 100%;
  height: 400px;
}

.w-widget-map label {
  width: auto;
  display: inline;
}

.w-widget-map img {
  max-width: inherit;
}

.w-widget-map .gm-style-iw {
  text-align: center;
}

.w-widget-map .gm-style-iw > button {
  display: none !important;
}

.w-widget-twitter {
  overflow: hidden;
}

.w-widget-twitter-count-shim {
  vertical-align: top;
  text-align: center;
  background: #fff;
  border: 1px solid #758696;
  border-radius: 3px;
  width: 28px;
  height: 20px;
  display: inline-block;
  position: relative;
}

.w-widget-twitter-count-shim * {
  pointer-events: none;
  -webkit-user-select: none;
  user-select: none;
}

.w-widget-twitter-count-shim .w-widget-twitter-count-inner {
  text-align: center;
  color: #999;
  font-family: serif;
  font-size: 15px;
  line-height: 12px;
  position: relative;
}

.w-widget-twitter-count-shim .w-widget-twitter-count-clear {
  display: block;
  position: relative;
}

.w-widget-twitter-count-shim.w--large {
  width: 36px;
  height: 28px;
}

.w-widget-twitter-count-shim.w--large .w-widget-twitter-count-inner {
  font-size: 18px;
  line-height: 18px;
}

.w-widget-twitter-count-shim:not(.w--vertical) {
  margin-left: 5px;
  margin-right: 8px;
}

.w-widget-twitter-count-shim:not(.w--vertical).w--large {
  margin-left: 6px;
}

.w-widget-twitter-count-shim:not(.w--vertical):before, .w-widget-twitter-count-shim:not(.w--vertical):after {
  content: " ";
  pointer-events: none;
  border: solid #0000;
  width: 0;
  height: 0;
  position: absolute;
  top: 50%;
  left: 0;
}

.w-widget-twitter-count-shim:not(.w--vertical):before {
  border-width: 4px;
  border-color: #75869600 #5d6c7b #75869600 #75869600;
  margin-top: -4px;
  margin-left: -9px;
}

.w-widget-twitter-count-shim:not(.w--vertical).w--large:before {
  border-width: 5px;
  margin-top: -5px;
  margin-left: -10px;
}

.w-widget-twitter-count-shim:not(.w--vertical):after {
  border-width: 4px;
  border-color: #fff0 #fff #fff0 #fff0;
  margin-top: -4px;
  margin-left: -8px;
}

.w-widget-twitter-count-shim:not(.w--vertical).w--large:after {
  border-width: 5px;
  margin-top: -5px;
  margin-left: -9px;
}

.w-widget-twitter-count-shim.w--vertical {
  width: 61px;
  height: 33px;
  margin-bottom: 8px;
}

.w-widget-twitter-count-shim.w--vertical:before, .w-widget-twitter-count-shim.w--vertical:after {
  content: " ";
  pointer-events: none;
  border: solid #0000;
  width: 0;
  height: 0;
  position: absolute;
  top: 100%;
  left: 50%;
}

.w-widget-twitter-count-shim.w--vertical:before {
  border-width: 5px;
  border-color: #5d6c7b #75869600 #75869600;
  margin-left: -5px;
}

.w-widget-twitter-count-shim.w--vertical:after {
  border-width: 4px;
  border-color: #fff #fff0 #fff0;
  margin-left: -4px;
}

.w-widget-twitter-count-shim.w--vertical .w-widget-twitter-count-inner {
  font-size: 18px;
  line-height: 22px;
}

.w-widget-twitter-count-shim.w--vertical.w--large {
  width: 76px;
}

.w-background-video {
  color: #fff;
  height: 500px;
  position: relative;
  overflow: hidden;
}

.w-background-video > video {
  object-fit: cover;
  z-index: -100;
  background-position: 50%;
  background-size: cover;
  width: 100%;
  height: 100%;
  margin: auto;
  position: absolute;
  inset: -100%;
}

.w-background-video > video::-webkit-media-controls-start-playback-button {
  -webkit-appearance: none;
  display: none !important;
}

.w-background-video--control {
  background-color: #0000;
  padding: 0;
  position: absolute;
  bottom: 1em;
  right: 1em;
}

.w-background-video--control > [hidden] {
  display: none !important;
}

.w-slider {
  text-align: center;
  clear: both;
  -webkit-tap-highlight-color: #0000;
  tap-highlight-color: #0000;
  background: #ddd;
  height: 300px;
  position: relative;
}

.w-slider-mask {
  z-index: 1;
  white-space: nowrap;
  height: 100%;
  display: block;
  position: relative;
  left: 0;
  right: 0;
  overflow: hidden;
}

.w-slide {
  vertical-align: top;
  white-space: normal;
  text-align: left;
  width: 100%;
  height: 100%;
  display: inline-block;
  position: relative;
}

.w-slider-nav {
  z-index: 2;
  text-align: center;
  -webkit-tap-highlight-color: #0000;
  tap-highlight-color: #0000;
  height: 40px;
  margin: auto;
  padding-top: 10px;
  position: absolute;
  inset: auto 0 0;
}

.w-slider-nav.w-round > div {
  border-radius: 100%;
}

.w-slider-nav.w-num > div {
  width: auto;
  height: auto;
  font-size: inherit;
  line-height: inherit;
  padding: .2em .5em;
}

.w-slider-nav.w-shadow > div {
  box-shadow: 0 0 3px #3336;
}

.w-slider-nav-invert {
  color: #fff;
}

.w-slider-nav-invert > div {
  background-color: #2226;
}

.w-slider-nav-invert > div.w-active {
  background-color: #222;
}

.w-slider-dot {
  cursor: pointer;
  background-color: #fff6;
  width: 1em;
  height: 1em;
  margin: 0 3px .5em;
  transition: background-color .1s, color .1s;
  display: inline-block;
  position: relative;
}

.w-slider-dot.w-active {
  background-color: #fff;
}

.w-slider-dot:focus {
  outline: none;
  box-shadow: 0 0 0 2px #fff;
}

.w-slider-dot:focus.w-active {
  box-shadow: none;
}

.w-slider-arrow-left, .w-slider-arrow-right {
  cursor: pointer;
  color: #fff;
  -webkit-tap-highlight-color: #0000;
  tap-highlight-color: #0000;
  -webkit-user-select: none;
  user-select: none;
  width: 80px;
  margin: auto;
  font-size: 40px;
  position: absolute;
  inset: 0;
  overflow: hidden;
}

.w-slider-arrow-left [class^="w-icon-"], .w-slider-arrow-right [class^="w-icon-"], .w-slider-arrow-left [class*=" w-icon-"], .w-slider-arrow-right [class*=" w-icon-"] {
  position: absolute;
}

.w-slider-arrow-left:focus, .w-slider-arrow-right:focus {
  outline: 0;
}

.w-slider-arrow-left {
  z-index: 3;
  right: auto;
}

.w-slider-arrow-right {
  z-index: 4;
  left: auto;
}

.w-icon-slider-left, .w-icon-slider-right {
  width: 1em;
  height: 1em;
  margin: auto;
  inset: 0;
}

.w-slider-aria-label {
  clip: rect(0 0 0 0);
  border: 0;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  position: absolute;
  overflow: hidden;
}

.w-slider-force-show {
  display: block !important;
}

.w-dropdown {
  text-align: left;
  z-index: 900;
  margin-left: auto;
  margin-right: auto;
  display: inline-block;
  position: relative;
}

.w-dropdown-btn, .w-dropdown-toggle, .w-dropdown-link {
  vertical-align: top;
  color: #222;
  text-align: left;
  white-space: nowrap;
  margin-left: auto;
  margin-right: auto;
  padding: 20px;
  text-decoration: none;
  position: relative;
}

.w-dropdown-toggle {
  -webkit-user-select: none;
  user-select: none;
  cursor: pointer;
  padding-right: 40px;
  display: inline-block;
}

.w-dropdown-toggle:focus {
  outline: 0;
}

.w-icon-dropdown-toggle {
  width: 1em;
  height: 1em;
  margin: auto 20px auto auto;
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
}

.w-dropdown-list {
  background: #ddd;
  min-width: 100%;
  display: none;
  position: absolute;
}

.w-dropdown-list.w--open {
  display: block;
}

.w-dropdown-link {
  color: #222;
  padding: 10px 20px;
  display: block;
}

.w-dropdown-link.w--current {
  color: #0082f3;
}

.w-dropdown-link:focus {
  outline: 0;
}

.w-lightbox-backdrop {
  cursor: auto;
  letter-spacing: normal;
  text-indent: 0;
  text-shadow: none;
  text-transform: none;
  visibility: visible;
  white-space: normal;
  word-break: normal;
  word-spacing: normal;
  word-wrap: normal;
  color: #fff;
  text-align: center;
  z-index: 2000;
  opacity: 0;
  -webkit-user-select: none;
  -moz-user-select: none;
  -webkit-tap-highlight-color: transparent;
  background: #000000e6;
  outline: 0;
  font-family: Helvetica Neue, Helvetica, Ubuntu, Segoe UI, Verdana, sans-serif;
  font-size: 17px;
  font-style: normal;
  font-weight: 300;
  line-height: 1.2;
  list-style: disc;
  position: fixed;
  inset: 0;
  -webkit-transform: translate(0);
}

.w-lightbox-backdrop, .w-lightbox-container {
  -webkit-overflow-scrolling: touch;
  height: 100%;
  overflow: auto;
}

.w-lightbox-content {
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.w-lightbox-view {
  opacity: 0;
  width: 100vw;
  height: 100vh;
  position: absolute;
}

.w-lightbox-view:before {
  content: "";
  height: 100vh;
}

.w-lightbox-group, .w-lightbox-group .w-lightbox-view, .w-lightbox-group .w-lightbox-view:before {
  height: 86vh;
}

.w-lightbox-frame, .w-lightbox-view:before {
  vertical-align: middle;
  display: inline-block;
}

.w-lightbox-figure {
  margin: 0;
  position: relative;
}

.w-lightbox-group .w-lightbox-figure {
  cursor: pointer;
}

.w-lightbox-img {
  width: auto;
  max-width: none;
  height: auto;
}

.w-lightbox-image {
  float: none;
  max-width: 100vw;
  max-height: 100vh;
  display: block;
}

.w-lightbox-group .w-lightbox-image {
  max-height: 86vh;
}

.w-lightbox-caption {
  text-align: left;
  text-overflow: ellipsis;
  white-space: nowrap;
  background: #0006;
  padding: .5em 1em;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  overflow: hidden;
}

.w-lightbox-embed {
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0;
}

.w-lightbox-control {
  cursor: pointer;
  background-position: center;
  background-repeat: no-repeat;
  background-size: 24px;
  width: 4em;
  transition: all .3s;
  position: absolute;
  top: 0;
}

.w-lightbox-left {
  background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9Ii0yMCAwIDI0IDQwIiB3aWR0aD0iMjQiIGhlaWdodD0iNDAiPjxnIHRyYW5zZm9ybT0icm90YXRlKDQ1KSI+PHBhdGggZD0ibTAgMGg1djIzaDIzdjVoLTI4eiIgb3BhY2l0eT0iLjQiLz48cGF0aCBkPSJtMSAxaDN2MjNoMjN2M2gtMjZ6IiBmaWxsPSIjZmZmIi8+PC9nPjwvc3ZnPg==");
  display: none;
  bottom: 0;
  left: 0;
}

.w-lightbox-right {
  background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9Ii00IDAgMjQgNDAiIHdpZHRoPSIyNCIgaGVpZ2h0PSI0MCI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoNDUpIj48cGF0aCBkPSJtMC0waDI4djI4aC01di0yM2gtMjN6IiBvcGFjaXR5PSIuNCIvPjxwYXRoIGQ9Im0xIDFoMjZ2MjZoLTN2LTIzaC0yM3oiIGZpbGw9IiNmZmYiLz48L2c+PC9zdmc+");
  display: none;
  bottom: 0;
  right: 0;
}

.w-lightbox-close {
  background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9Ii00IDAgMTggMTciIHdpZHRoPSIxOCIgaGVpZ2h0PSIxNyI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoNDUpIj48cGF0aCBkPSJtMCAwaDd2LTdoNXY3aDd2NWgtN3Y3aC01di03aC03eiIgb3BhY2l0eT0iLjQiLz48cGF0aCBkPSJtMSAxaDd2LTdoM3Y3aDd2M2gtN3Y3aC0zdi03aC03eiIgZmlsbD0iI2ZmZiIvPjwvZz48L3N2Zz4=");
  background-size: 18px;
  height: 2.6em;
  right: 0;
}

.w-lightbox-strip {
  white-space: nowrap;
  padding: 0 1vh;
  line-height: 0;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  overflow: auto hidden;
}

.w-lightbox-item {
  box-sizing: content-box;
  cursor: pointer;
  width: 10vh;
  padding: 2vh 1vh;
  display: inline-block;
  -webkit-transform: translate3d(0, 0, 0);
}

.w-lightbox-active {
  opacity: .3;
}

.w-lightbox-thumbnail {
  background: #222;
  height: 10vh;
  position: relative;
  overflow: hidden;
}

.w-lightbox-thumbnail-image {
  position: absolute;
  top: 0;
  left: 0;
}

.w-lightbox-thumbnail .w-lightbox-tall {
  width: 100%;
  top: 50%;
  transform: translate(0, -50%);
}

.w-lightbox-thumbnail .w-lightbox-wide {
  height: 100%;
  left: 50%;
  transform: translate(-50%);
}

.w-lightbox-spinner {
  box-sizing: border-box;
  border: 5px solid #0006;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  margin-top: -20px;
  margin-left: -20px;
  animation: .8s linear infinite spin;
  position: absolute;
  top: 50%;
  left: 50%;
}

.w-lightbox-spinner:after {
  content: "";
  border: 3px solid #0000;
  border-bottom-color: #fff;
  border-radius: 50%;
  position: absolute;
  inset: -4px;
}

.w-lightbox-hide {
  display: none;
}

.w-lightbox-noscroll {
  overflow: hidden;
}

@media (min-width: 768px) {
  .w-lightbox-content {
    height: 96vh;
    margin-top: 2vh;
  }

  .w-lightbox-view, .w-lightbox-view:before {
    height: 96vh;
  }

  .w-lightbox-group, .w-lightbox-group .w-lightbox-view, .w-lightbox-group .w-lightbox-view:before {
    height: 84vh;
  }

  .w-lightbox-image {
    max-width: 96vw;
    max-height: 96vh;
  }

  .w-lightbox-group .w-lightbox-image {
    max-width: 82.3vw;
    max-height: 84vh;
  }

  .w-lightbox-left, .w-lightbox-right {
    opacity: .5;
    display: block;
  }

  .w-lightbox-close {
    opacity: .8;
  }

  .w-lightbox-control:hover {
    opacity: 1;
  }
}

.w-lightbox-inactive, .w-lightbox-inactive:hover {
  opacity: 0;
}

.w-richtext:before, .w-richtext:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-richtext:after {
  clear: both;
}

.w-richtext[contenteditable="true"]:before, .w-richtext[contenteditable="true"]:after {
  white-space: initial;
}

.w-richtext ol, .w-richtext ul {
  overflow: hidden;
}

.w-richtext .w-richtext-figure-selected.w-richtext-figure-type-video div:after, .w-richtext .w-richtext-figure-selected[data-rt-type="video"] div:after, .w-richtext .w-richtext-figure-selected.w-richtext-figure-type-image div, .w-richtext .w-richtext-figure-selected[data-rt-type="image"] div {
  outline: 2px solid #2895f7;
}

.w-richtext figure.w-richtext-figure-type-video > div:after, .w-richtext figure[data-rt-type="video"] > div:after {
  content: "";
  display: none;
  position: absolute;
  inset: 0;
}

.w-richtext figure {
  max-width: 60%;
  position: relative;
}

.w-richtext figure > div:before {
  cursor: default !important;
}

.w-richtext figure img {
  width: 100%;
}

.w-richtext figure figcaption.w-richtext-figcaption-placeholder {
  opacity: .6;
}

.w-richtext figure div {
  color: #0000;
  font-size: 0;
}

.w-richtext figure.w-richtext-figure-type-image, .w-richtext figure[data-rt-type="image"] {
  display: table;
}

.w-richtext figure.w-richtext-figure-type-image > div, .w-richtext figure[data-rt-type="image"] > div {
  display: inline-block;
}

.w-richtext figure.w-richtext-figure-type-image > figcaption, .w-richtext figure[data-rt-type="image"] > figcaption {
  caption-side: bottom;
  display: table-caption;
}

.w-richtext figure.w-richtext-figure-type-video, .w-richtext figure[data-rt-type="video"] {
  width: 60%;
  height: 0;
}

.w-richtext figure.w-richtext-figure-type-video iframe, .w-richtext figure[data-rt-type="video"] iframe {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

.w-richtext figure.w-richtext-figure-type-video > div, .w-richtext figure[data-rt-type="video"] > div {
  width: 100%;
}

.w-richtext figure.w-richtext-align-center {
  clear: both;
  margin-left: auto;
  margin-right: auto;
}

.w-richtext figure.w-richtext-align-center.w-richtext-figure-type-image > div, .w-richtext figure.w-richtext-align-center[data-rt-type="image"] > div {
  max-width: 100%;
}

.w-richtext figure.w-richtext-align-normal {
  clear: both;
}

.w-richtext figure.w-richtext-align-fullwidth {
  text-align: center;
  clear: both;
  width: 100%;
  max-width: 100%;
  margin-left: auto;
  margin-right: auto;
  display: block;
}

.w-richtext figure.w-richtext-align-fullwidth > div {
  padding-bottom: inherit;
  display: inline-block;
}

.w-richtext figure.w-richtext-align-fullwidth > figcaption {
  display: block;
}

.w-richtext figure.w-richtext-align-floatleft {
  float: left;
  clear: none;
  margin-right: 15px;
}

.w-richtext figure.w-richtext-align-floatright {
  float: right;
  clear: none;
  margin-left: 15px;
}

.w-nav {
  z-index: 1000;
  background: #ddd;
  position: relative;
}

.w-nav:before, .w-nav:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-nav:after {
  clear: both;
}

.w-nav-brand {
  float: left;
  color: #333;
  text-decoration: none;
  position: relative;
}

.w-nav-link {
  vertical-align: top;
  color: #222;
  text-align: left;
  margin-left: auto;
  margin-right: auto;
  padding: 20px;
  text-decoration: none;
  display: inline-block;
  position: relative;
}

.w-nav-link.w--current {
  color: #0082f3;
}

.w-nav-menu {
  float: right;
  position: relative;
}

[data-nav-menu-open] {
  text-align: center;
  background: #c8c8c8;
  min-width: 200px;
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  overflow: visible;
  display: block !important;
}

.w--nav-link-open {
  display: block;
  position: relative;
}

.w-nav-overlay {
  width: 100%;
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  overflow: hidden;
}

.w-nav-overlay [data-nav-menu-open] {
  top: 0;
}

.w-nav[data-animation="over-left"] .w-nav-overlay {
  width: auto;
}

.w-nav[data-animation="over-left"] .w-nav-overlay, .w-nav[data-animation="over-left"] [data-nav-menu-open] {
  z-index: 1;
  top: 0;
  right: auto;
}

.w-nav[data-animation="over-right"] .w-nav-overlay {
  width: auto;
}

.w-nav[data-animation="over-right"] .w-nav-overlay, .w-nav[data-animation="over-right"] [data-nav-menu-open] {
  z-index: 1;
  top: 0;
  left: auto;
}

.w-nav-button {
  float: right;
  cursor: pointer;
  -webkit-tap-highlight-color: #0000;
  tap-highlight-color: #0000;
  -webkit-user-select: none;
  user-select: none;
  padding: 18px;
  font-size: 24px;
  display: none;
  position: relative;
}

.w-nav-button:focus {
  outline: 0;
}

.w-nav-button.w--open {
  color: #fff;
  background-color: #c8c8c8;
}

.w-nav[data-collapse="all"] .w-nav-menu {
  display: none;
}

.w-nav[data-collapse="all"] .w-nav-button, .w--nav-dropdown-open, .w--nav-dropdown-toggle-open {
  display: block;
}

.w--nav-dropdown-list-open {
  position: static;
}

@media screen and (max-width: 991px) {
  .w-nav[data-collapse="medium"] .w-nav-menu {
    display: none;
  }

  .w-nav[data-collapse="medium"] .w-nav-button {
    display: block;
  }
}

@media screen and (max-width: 767px) {
  .w-nav[data-collapse="small"] .w-nav-menu {
    display: none;
  }

  .w-nav[data-collapse="small"] .w-nav-button {
    display: block;
  }

  .w-nav-brand {
    padding-left: 10px;
  }
}

@media screen and (max-width: 479px) {
  .w-nav[data-collapse="tiny"] .w-nav-menu {
    display: none;
  }

  .w-nav[data-collapse="tiny"] .w-nav-button {
    display: block;
  }
}

.w-tabs {
  position: relative;
}

.w-tabs:before, .w-tabs:after {
  content: " ";
  grid-area: 1 / 1 / 2 / 2;
  display: table;
}

.w-tabs:after {
  clear: both;
}

.w-tab-menu {
  position: relative;
}

.w-tab-link {
  vertical-align: top;
  text-align: left;
  cursor: pointer;
  color: #222;
  background-color: #ddd;
  padding: 9px 30px;
  text-decoration: none;
  display: inline-block;
  position: relative;
}

.w-tab-link.w--current {
  background-color: #c8c8c8;
}

.w-tab-link:focus {
  outline: 0;
}

.w-tab-content {
  display: block;
  position: relative;
  overflow: hidden;
}

.w-tab-pane {
  display: none;
  position: relative;
}

.w--tab-active {
  display: block;
}

@media screen and (max-width: 479px) {
  .w-tab-link {
    display: block;
  }
}

.w-ix-emptyfix:after {
  content: "";
}

@keyframes spin {
  0% {
    transform: rotate(0);
  }

  100% {
    transform: rotate(360deg);
  }
}

.w-dyn-empty {
  background-color: #ddd;
  padding: 10px;
}

.w-dyn-hide, .w-dyn-bind-empty, .w-condition-invisible {
  display: none !important;
}

.wf-layout-layout {
  display: grid;
}

.w-layout-grid {
  grid-row-gap: 16px;
  grid-column-gap: 16px;
  grid-template-rows: auto auto;
  grid-template-columns: 1fr 1fr;
  grid-auto-columns: 1fr;
  display: grid;
}











h1 {
  margin-top: 0;
  margin-bottom: 0;
  font-size: 38px;
  font-weight: 700;
  line-height: 44px;
}

h2 {
  margin-top: 0;
  margin-bottom: 0;
  font-size: 32px;
  font-weight: 700;
  line-height: 36px;
}

h3 {
  margin-top: 0;
  margin-bottom: 0;
  font-size: 24px;
  font-weight: 700;
  line-height: 30px;
}

h4 {
  margin-top: 0;
  margin-bottom: 0;
  font-size: 18px;
  font-weight: 700;
  line-height: 24px;
}

a {
  text-decoration: none;
}

strong {
  font-weight: 600;
}

.hs_main-wrapper {
  color: #172349;
  flex-flow: column;
  width: 100%;
  height: calc(100dvh - 214.5px);
  max-height: calc(100vh - 214.5px);
  font-family: Sohne, Arial, sans-serif;
  font-size: 14px;
  line-height: 1.2;
  display: flex;
  position: relative;
  overflow: clip;
}

.hs_quiz_section {
  flex-flow: column;
  flex: 1;
  padding-top: 10px;
  padding-bottom: 10px;
  display: flex;
  position: relative;
}

.hs_quiz_nav {
  color: #fff;
  background-color: #172349;
  width: 100%;
  padding: 16px;
}

.hs_quiz_nav_logo {
  aspect-ratio: 114 / 14;
  width: 114px;
}

.hs_quiz_nav_container {
  grid-column-gap: 1rem;
  grid-row-gap: 1rem;
  justify-content: space-between;
  align-items: center;
  max-width: 1360px;
  margin-left: auto;
  margin-right: auto;
  display: flex;
}

.hs_quiz_nav_powered-by {
  aspect-ratio: 107 / 30;
  object-fit: contain;
  width: 107px;
}

.hs_quiz_container {
  z-index: 4;
  width: 100%;
  max-width: 1440px;
  height: 100%;
  margin-left: auto;
  margin-right: auto;
  padding-left: 16px;
  padding-right: 16px;
}

.hs_quiz_section_bg {
  z-index: -1;
  pointer-events: none;
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_section_bg_overlay {
  z-index: 2;
  background-color: #0003;
  width: 100%;
  height: 100%;
  position: absolute;
}

.hs_quiz_section_bg_img {
  z-index: 1;
  object-fit: cover;
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_intro_wrap {
  flex-flow: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  display: flex;
}

.hs_quiz_intro_card {
  background-color: #fff;
  border-radius: 10px;
  width: 100%;
  max-width: 1153px;
  margin-left: auto;
  margin-right: auto;
  padding: 40px;
  box-shadow: 0 4px 7px 3px #00000026;
}

.hs_quiz_intro_title {
  text-transform: uppercase;
  font-size: 60px;
  font-weight: 400;
  line-height: 1.3;
}

.hs_quiz_intro_title-medium {
  font-weight: 500;
}

.hs_quiz_intro_title-wrap {
  padding-top: 20px;
  padding-bottom: 20px;
}

.hs_quiz_intro_content-wrap {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  flex-flow: row;
  max-height: 301px;
  margin-top: 20px;
  display: flex;
}

.hs_quiz_intro_content-info-wrap {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  background-color: #f3f4f8;
  border-radius: 8px;
  flex-flow: column;
  width: 48.6557%;
  padding: 10px;
  display: flex;
  overflow: auto;
}

.hs_quiz_intro_cta-col {
  grid-column-gap: 13px;
  grid-row-gap: 13px;
  flex-flow: column;
  flex: 1;
  width: 20.8153%;
  display: flex;
}

.hs_quiz_intro_info-text {
  font-size: 18px;
  line-height: 1.39;
}

.hs_quiz_intro_col-title {
  font-weight: 500;
}

.hs_quiz_intro_col-desc {
  margin-bottom: 4px;
  font-size: 15px;
  line-height: 1.39;
}

.hs_quiz_button-navy {
  grid-column-gap: 1.25rem;
  grid-row-gap: 1.25rem;
  color: #fff;
  background-color: #172349;
  border: 2px solid #172349;
  border-radius: 5px;
  justify-content: center;
  align-items: center;
  min-height: 3.5rem;
  padding: 16px;
  font-size: 16px;
  font-weight: 500;
  line-height: 1.2;
  transition: color .4s cubic-bezier(.165, .84, .44, 1), background-color .4s cubic-bezier(.165, .84, .44, 1);
  display: flex;
}

.hs_quiz_button-navy:hover {
  color: #172349;
  background-color: #f3f4f8;
}

.hs_main-styles {
  display: none;
}

.hs_quiz_button-icon {
  width: 1.5rem;
}

.hs_mt-auto {
  margin-top: auto;
}

.hs_quiz_intro_list {
  color: #172349;
  padding-left: 15px;
}

.hs_quiz_intro_list.mt-5 {
  margin-top: 5px;
}

.hs_quiz_persona_card {
  background-color: #fff;
  border-radius: 10px;
  width: 100%;
  max-width: 993px;
  margin-left: auto;
  margin-right: auto;
  padding: 20px 40px;
  box-shadow: 0 4px 7px 3px #00000026;
}

.hs_quiz_persona_title {
  text-align: center;
  font-size: 16px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_persona_tab {
  flex-flow: column;
  justify-content: flex-start;
  align-items: center;
  margin-top: 16px;
  display: flex;
}

.hs_quiz_persona_tab-link {
  color: #172349;
  text-align: center;
  background-color: #0000;
  border-radius: 100vw;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  min-width: 250px;
  font-size: 16px;
  font-weight: 500;
  line-height: 1.2;
  transition: background-color .4s cubic-bezier(.165, .84, .44, 1);
  display: inline-flex;
}

.hs_quiz_persona_tab-link.w--current {
  background-color: #c0de7a;
}

.hs_quiz_persona_tab-menu {
  background-color: #f3f4f8;
  border-radius: 100vw;
  font-size: 16px;
  line-height: 1.2;
  display: inline-flex;
  overflow: hidden;
}

.hs_quiz_persona_tab-content {
  width: 100%;
  margin-top: 16px;
}

.hs_quiz_persona_tab-info {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  background-color: #f3f4f8;
  border-radius: 8px;
  flex-flow: column;
  width: 100%;
  height: 223px;
  max-height: 223px;
  padding: 10px;
  display: flex;
  overflow: auto;
}

.hs_quiz_persona_tab-info-text {
  font-size: 15px;
  line-height: 1.39;
}

.hs_text-weight-medium {
  font-weight: 500;
}

.hs_quiz_button-blue {
  color: #000;
  text-align: center;
  background-color: #8bcff2;
  border-radius: 5px;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  min-width: 337px;
  padding: 16px;
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
  display: flex;
}

.hs_quiz_tab-button-wrap {
  flex-flow: column;
  justify-content: center;
  align-items: center;
  margin-top: 16px;
  display: flex;
}

.hs_quiz_button-green {
  color: #000;
  text-align: center;
  background-color: #c0de7a;
  border-radius: 5px;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  min-width: 337px;
  padding: 16px;
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
  display: flex;
}

.hs_quiz_header {
  z-index: 99;
  flex-flow: column;
  justify-content: flex-end;
  align-items: center;
  height: 66px;
  display: flex;
  position: relative;
}

.hs_quiz_header_cta {
  z-index: 6;
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  color: #172349;
  background-color: #fff;
  border: 2px solid #172349;
  border-radius: 10px;
  justify-content: center;
  align-items: center;
  padding: 10px 42px;
  text-decoration: none;
  transition-property: color, background-color;
  transition-duration: .4s, .4s;
  transition-timing-function: cubic-bezier(.165, .84, .44, 1), cubic-bezier(.165, .84, .44, 1);
  display: flex;
  position: absolute;
  transform: translateY(calc(100% + 10px));
}

.hs_quiz_header_cta:hover {
  color: #f3f4f8;
  background-color: #172349;
}

.hs_quiz_header_cta-text {
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_assessment_wrapper {
  flex-flow: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  display: flex;
}

.hs_quiz_assessment_form {
  background-color: #0000;
}

.hs_hide {
  display: none !important;
}

.hs_quiz_assessment_card {
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  background-color: #fff;
  border-radius: 10px;
  flex-flow: column;
  width: 100%;
  max-width: 530px;
  padding: 10px;
  display: flex;
  box-shadow: 0 4px 7.7px 3px #00000026;
}

.hs_quiz_assessment_header {
  grid-column-gap: 8px;
  grid-row-gap: 8px;
  justify-content: flex-start;
  align-items: center;
  display: flex;
}

.hs_quiz_assessment_question {
  font-size: 14px;
  font-weight: 500;
  line-height: 1.3;
}

.hs_quiz_assessment_progress-bar {
  background-color: #f3f4f8;
  border-radius: 100vw;
  flex: none;
  width: 64%;
  height: .625rem;
}

.hs_quiz_assessment_progress-fill {
  background-color: #67b4f8;
  border-radius: 100vw;
  width: 16.6667%;
  height: 100%;
  transition: width .8s cubic-bezier(.165, .84, .44, 1);
}

.hs_quiz_assessment_sector {
  grid-column-gap: 8px;
  grid-row-gap: 8px;
  justify-content: space-between;
  align-items: center;
  display: flex;
}

.hs_quiz_assessment_sector-label {
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_assessment_sector-img {
  aspect-ratio: 90 / 50;
  object-fit: contain;
  flex: none;
  width: 90px;
  height: auto;
}

.hs_quiz_assessment_sector-label-wrap {
  flex: 1;
}

.hs_quiz_assessment_q-block {
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_q-main {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_text {
  font-size: 15px;
  font-weight: 400;
  line-height: 1.3;
}

.hs_quiz_assessment_q-options {
  grid-column-gap: 6px;
  grid-row-gap: 6px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_q-radio {
  cursor: pointer;
  border: 1px solid #e9e9ed;
  border-radius: 10px;
  justify-content: flex-start;
  align-items: center;
  margin-bottom: 0;
  padding: 12px 14px;
  transition: background-color .4s cubic-bezier(.165, .84, .44, 1), border-color .4s cubic-bezier(.165, .84, .44, 1);
  display: flex;
}

.hs_quiz_assessment_q-radio-label {
  font-size: 15px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_assessment_q-radio-btn {
  border: 1px solid #000;
  border-radius: 100vw;
  flex: none;
  width: 15px;
  height: 15px;
  margin-top: 0;
  margin-left: 0;
  margin-right: 14px;
  transition: border-color .4s cubic-bezier(.165, .84, .44, 1), background-color .4s cubic-bezier(.165, .84, .44, 1);
}

.hs_quiz_assessment_qa-label {
  margin-right: 4px;
}

.hs_quiz_assessment_a-block {
  grid-column-gap: 20px;
  grid-row-gap: 20px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_a-img-wrap {
  flex-flow: column;
  justify-content: center;
  align-items: center;
  display: flex;
}

.hs_quiz_assessment_a-img {
  aspect-ratio: 90 / 70;
  object-fit: contain;
  width: 90px;
}

.hs_quiz_assessment_a-inner-block {
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_a-next-wrap {
  flex-flow: column;
  display: flex;
}

.hs_quiz_assessment_a-next-btn {
  color: #000;
  text-align: center;
  background-color: #8bcff2;
  border-radius: 5px;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  padding: 16px;
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
  display: flex;
}

.hs_quiz_assessment_agriculture.show {
  max-height: 50vh;
  overflow: auto;
}

.hs_quiz_assessment_set {
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  flex-flow: column;
  display: flex;
}

.hs_quiz_outcome_wrapper {
  flex-flow: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  display: flex;
}

.hs_quiz_outcome_card {
  color: #172349;
  background-color: #fff;
  border-radius: 10px;
  max-width: 993px;
  padding: 20px 40px;
  box-shadow: 0 4px 7.7px 3px #00000026;
}

.hs_quiz_outcome_title {
  text-align: center;
  font-size: 24px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_outcome_content {
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  flex-flow: column;
  height: 100%;
  margin-top: 16px;
  display: flex;
}

.hs_quiz_outcome_accordion {
  background-color: #e7ffcb;
  border: 2px solid #81c24e;
  border-radius: 5px;
  flex: none;
  min-height: 0;
  max-height: 290px;
  padding-left: 10px;
  padding-right: 10px;
  transition: min-height .3s cubic-bezier(.77, 0, .175, 1);
  overflow: auto;
}

.hs_quiz_outcome_accordion.is-2 {
  background-color: #f6f0bc;
  border-color: #ec8703;
}

.hs_quiz_outcome_accordion.is-3 {
  background-color: #e94c521a;
  border-color: #e94c52;
}

.hs_quiz_outcome_accordion_trigger {
  grid-column-gap: 24px;
  grid-row-gap: 24px;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding-top: 12px;
  padding-bottom: 12px;
  display: flex;
}

.hs_quiz_outcome_accordion_content {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  grid-template-rows: 0fr;
  grid-auto-columns: 1fr;
  width: 92.552%;
  transition-property: all;
  transition-duration: .6s;
  transition-timing-function: cubic-bezier(.165, .84, .44, 1);
  display: none;
}

.hs_quiz_outcome_accordion_title {
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
}

.hs_quiz_outcome_accordion_para {
  margin-bottom: 0;
  padding-top: 4px;
  padding-bottom: 10px;
}

.hs_quiz_outcome_accordion_icon {
  background-color: #172349;
  border-radius: 100vw;
  flex-flow: column;
  flex: none;
  justify-content: center;
  align-items: center;
  width: 24px;
  height: 24px;
  display: none;
}

.hs_quiz_outcome_accordion_trigger-wrap {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  flex-flow: column;
  width: 92.552%;
  display: flex;
}

.hs_quiz_outcome_accordion_line {
  background-color: #fff;
  border-radius: 100vw;
  width: 12px;
  height: 2px;
  transition: width .4s cubic-bezier(.165, .84, .44, 1);
}

.hs_quiz_outcome_accordion_line.is-2 {
  position: absolute;
  transform: rotate(90deg);
}

.hs_quiz_outcome_accordion_content-inner {
  overflow: hidden;
}

.hs_quiz_outcome_info {
  grid-column-gap: 16px;
  grid-row-gap: 16px;
  background-color: #f3f4f8;
  border-radius: 8px;
  flex-flow: column;
  height: 201px;
  padding: 10px;
  display: flex;
  overflow: auto;
}

.hs_quiz_outcome_info-text {
  font-size: 15px;
  line-height: 1.39;
}

.hs_quiz_outcome_button-wrap {
  grid-column-gap: 8px;
  grid-row-gap: 8px;
  flex-flow: row;
  justify-content: center;
  display: flex;
}

.hs_quiz_button-outlined {
  color: #000;
  text-align: center;
  border: 2px solid #172349;
  border-radius: 5px;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  min-width: 21.0625rem;
  padding: 16px;
  font-size: 18px;
  font-weight: 500;
  line-height: 1.2;
  transition: color .4s cubic-bezier(.165, .84, .44, 1), background-color .4s cubic-bezier(.165, .84, .44, 1);
  display: flex;
}

.hs_quiz_button-outlined:hover {
  color: #fff;
  background-color: #172349;
}

.hs_quiz_button-outlined.is-intro {
  min-width: 0;
  font-size: 16px;
  line-height: 1.2;
}

.hs_quiz_outcome_healthcare.show, .hs_quiz_outcome_agriculture.show {
  max-height: 50vh;
  overflow: auto;
}

.hs_confetti-canvas {
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_mobile-cta {
  pointer-events: none;
  visibility: hidden;
  display: none;
}

.hs_quiz_outcome_fixed-wrap {
  grid-column-gap: .625rem;
  grid-row-gap: .625rem;
  flex-flow: column;
  max-height: 290px;
  display: flex;
}

.hs_quiz_section_bg_img-wrap {
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_section_bg_img-overlay {
  z-index: 2;
  background-color: #0003;
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_section_bg_img-overlay.is-1 {
  -webkit-backdrop-filter: blur(5px);
  backdrop-filter: blur(5px);
  background-color: #ffffff1a;
}

.hs_quiz_section_bg_img-overlay.is-3 {
  background-color: #0003;
}

.hs_quiz_section_bg_img-overlay.is-outcome {
  -webkit-backdrop-filter: blur(3px);
  backdrop-filter: blur(3px);
}

.hs_quiz_section_bg_img-overlay.is-5 {
  background-color: #0000;
}

.hs_quiz_assessment_a-overlay {
  z-index: -1;
  -webkit-backdrop-filter: blur(3px);
  backdrop-filter: blur(3px);
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0%;
}

.hs_quiz_nav_link {
  color: #fff;
  width: 100%;
  max-width: 7.125rem;
  transition: color .4s cubic-bezier(.165, .84, .44, 1);
}

.hs_quiz_button-white {
  color: #fff;
  text-align: center;
  cursor: pointer;
  border: .75px solid #ffffff4d;
  border-radius: 5px;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  padding: 10px 12px;
  font-size: 12px;
  font-weight: 500;
  line-height: 1.2;
  transition: border-color .4s cubic-bezier(.165, .84, .44, 1);
  display: flex;
}

.hs_quiz_button-white:hover {
  border-color: #ffffffa3;
}

.hs_quiz_nav_cta-wrap {
  grid-column-gap: 24px;
  grid-row-gap: 24px;
  justify-content: flex-end;
  align-items: center;
  width: 100%;
  max-width: 7.125rem;
  display: flex;
}

.hs_link_outcome {
  color: #172349;
  text-decoration: underline;
}

.hs_link_outcome:hover {
  text-decoration: none;
}

@media screen and (max-width: 991px) {
  strong {
    font-weight: 500;
  }

  .hs_quiz_intro_card {
    padding: 16px 10px;
  }

  .hs_quiz_intro_title {
    font-size: 40px;
  }

  .hs_quiz_intro_title-wrap {
    padding-top: 0;
    padding-bottom: 0;
  }

  .hs_quiz_intro_content-wrap {
    grid-column-gap: 8px;
    grid-row-gap: 8px;
    flex-flow: column;
    margin-top: 8px;
  }

  .hs_quiz_intro_content-info-wrap {
    width: 100%;
  }

  .hs_quiz_intro_cta-col {
    display: contents;
  }

  .hs_quiz_intro_col-title, .hs_quiz_intro_col-content {
    display: none;
  }

  .hs_quiz_persona_card {
    padding: 16px 10px;
  }

  .hs_quiz_persona_tab {
    margin-top: 10px;
  }

  .hs_quiz_persona_tab-link {
    min-width: 0;
    padding: 10px 20px;
  }

  .hs_quiz_persona_tab-link.w--current {
    min-width: 0;
  }

  .hs_quiz_button-blue {
    min-width: 0;
    font-size: 16px;
  }

  .hs_quiz_button-blue.is-m-18 {
    width: 100%;
    font-size: 18px;
  }

  .hs_quiz_button-green {
    min-width: 0;
    font-size: 16px;
  }

  .hs_quiz_button-green.is-m-18 {
    width: 100%;
    font-size: 18px;
  }

  .hs_quiz_header_cta {
    pointer-events: none;
    display: none;
  }

  .hs_quiz_header_cta.is-mobile {
    pointer-events: auto;
    border-top-width: 2px;
    border-bottom-width: 0;
    border-radius: 5px 5px 0 0;
    padding: 8px 24px;
    display: flex;
    position: static;
    top: auto;
    bottom: 0%;
    transform: translate(0);
  }

  .hs_quiz_header_cta-text {
    font-size: 15px;
  }

  .hs_quiz_outcome_card {
    max-width: 300px;
    padding: 10px;
  }

  .hs_quiz_outcome_title {
    font-size: 16px;
  }

  .hs_quiz_outcome_content {
    margin-top: 10px;
  }

  .hs_quiz_outcome_accordion {
    max-height: 262px;
  }

  .hs_quiz_outcome_button-wrap {
    flex-flow: column;
  }

  .hs_quiz_button-outlined {
    min-width: 0;
    font-size: 16px;
  }

  .hs_quiz_outcome_healthcare.show {
    overflow: auto;
  }

  .hs_quiz_mobile-cta {
    pointer-events: auto;
    visibility: visible;
    flex-flow: column;
    justify-content: flex-start;
    align-items: center;
    display: flex;
    margin-bottom: -10px !important;
  }

  .hs_quiz_outcome_fixed-wrap {
    max-height: 262px;
    overflow: hidden;
  }

  .hs_quiz_button-white {
    min-width: 0;
  }

  .hs_quiz_button-white.is-m-18 {
    width: 100%;
    font-size: 18px;
  }
}

@media screen and (max-width: 767px) {
  .hs_main-wrapper {
    height: calc(100dvh - 178.5px);
    max-height: calc(100vh - 178.5px);
  }

  .hs_quiz_intro_title {
    font-size: 24px;
    font-weight: 600;
  }

  .hs_quiz_intro_title-medium {
    font-weight: 600;
  }

  .hs_quiz_intro_info-text {
    font-size: 15px;
  }

  .hs_quiz_persona_tab-link {
    padding-left: 15px;
    padding-right: 15px;
  }

  .hs_quiz_tab-button-wrap {
    align-items: stretch;
  }

  .hs_quiz_assessment_text, .hs_quiz_assessment_q-radio-label {
    font-size: 14px;
  }

  .hs_quiz_assessment_qa-label {
    margin-right: 4px;
  }

  .hs_quiz_outcome_accordion_content {
    width: 88%;
  }

  .hs_quiz_outcome_accordion_title {
    font-size: 16px;
  }

  .hs_quiz_outcome_accordion_para {
    font-size: 15px;
    line-height: 1.39;
  }
}


.w-webflow-badge{display:none !important;}
`;
    document.head.appendChild(layoutStyle);

    // Set Webflow page/site attributes on <html> (required for IX2 interactions)
    var wfAttrs = {"data-wf-page":"6a9a9a6f2876b5a5a344309d","data-wf-site":"6a9a9a6d2876b5a5a344308c","data-wf-domain":"bayer-quiz.webflow.io"};
    Object.keys(wfAttrs).forEach(function(key) {
      document.documentElement.setAttribute(key, wfAttrs[key]);
    });

    document.documentElement.classList.add('w-mod-js');

    var container = document.createElement('div');
    container.className = 'hlabs-page-root';
    container.innerHTML = `<div class="hs_main-wrapper"><div class="hs_main-styles w-embed"></div><div class="hs_quiz_header"><div class="hs_quiz_nav"><div class="hs_quiz_nav_container"><div class="hs_quiz_nav_link"><svg xmlns="http://www.w3.org/2000/svg" width="100%" viewBox="0 0 115 14" fill="none" class="hs_quiz_nav_logo"><path d="M100.949 13.8411L106.686 0.00380097H108.286L114.023 13.8411H112.779L111.302 10.3054H103.67L102.193 13.8411H100.949ZM110.854 9.21507L107.481 1.10435L104.118 9.21507H110.854Z" fill="currentColor"></path><path d="M78.7287 13.888V0.00380097H84.587C87.2455 0.00380097 88.973 1.51691 88.973 3.93999C88.973 5.9439 87.736 7.365 85.7424 7.76377L89.3103 13.8879H87.8688L84.444 7.86599H79.9554V13.8879L78.7287 13.888ZM79.9559 6.74145H84.526C86.6216 6.74145 87.7155 5.6168 87.7155 3.94005C87.7155 2.23265 86.6216 1.12845 84.526 1.12845H79.9559V6.74145Z" fill="currentColor"></path><path d="M92.8182 8.07785C91.7339 7.02607 91.216 6.22516 91.216 5.35946C91.216 3.63622 92.6398 2.68153 94.2825 2.68153C96.0219 2.68153 97.3079 3.70904 97.3079 5.18956C97.3079 6.54063 95.9895 7.56814 94.3064 8.27202L97.6076 11.4111C98.085 10.78 98.2788 9.98717 98.2788 9.20238C98.2788 8.8464 98.2463 8.49041 98.1978 8.1587H99.0878C99.1203 8.47423 99.1363 8.7978 99.1363 9.1053C99.1363 10.157 98.9021 11.1522 98.2383 12.0097L100.164 13.83H98.8935L97.664 12.665C96.9683 13.2718 95.9325 14 94.1121 14C91.7903 14 90.1238 12.8107 90.1238 11.0874C90.1238 9.63119 91.4103 8.74937 92.8182 8.07785ZM91.1026 11.0713C91.1026 12.301 92.3971 13.11 94.1366 13.11C95.5445 13.11 96.4424 12.6085 97.0247 12.0583L93.4164 8.64413C92.1385 9.2752 91.1026 9.84149 91.1026 11.0713ZM93.6916 7.69759C95.3747 6.96134 96.3535 6.16037 96.3535 5.22192C96.3535 4.19447 95.4881 3.55531 94.2984 3.55531C93.2387 3.55531 92.1709 4.1216 92.1709 5.31901C92.1709 6.2737 92.8506 6.89662 93.6916 7.69759Z" fill="currentColor"></path><path d="M58.9742 11.8502C57.7225 11.8502 56.6012 11.3814 55.6417 10.4566C54.6788 9.53474 54.1905 8.34069 54.1905 6.90763C54.1905 5.54157 54.6623 4.37823 55.5921 3.44975C56.5214 2.5089 57.6763 2.03178 59.0255 2.03178C60.2362 2.03178 61.2852 2.41426 62.1507 3.16692C62.4578 2.57864 62.8441 2.02215 63.3095 1.49888C62.0857 0.520036 60.6596 0.0181903 59.0517 0.0181903C57.1379 0.0181903 55.4736 0.698884 54.1062 2.0413C52.7354 3.38445 52.0402 5.01466 52.0402 6.88644C52.0402 9.01364 52.8027 10.7592 54.3068 12.0747C55.6742 13.2619 57.2365 13.8639 58.9492 13.8639C60.5895 13.8639 62.0401 13.3704 63.2793 12.4103C62.8184 11.8917 62.4333 11.3253 62.1284 10.7155C61.2493 11.4675 60.1907 11.8502 58.9742 11.8502Z" fill="currentColor"></path><path d="M17.7205 13.9084C16.0017 13.9084 14.435 13.3047 13.0639 12.114C11.5553 10.7944 10.7904 9.04378 10.7904 6.91082C10.7904 5.03466 11.4877 3.39978 12.8631 2.05173C14.2344 0.705328 15.9031 0.0226972 17.8228 0.0226972C19.7222 0.0226972 21.3736 0.711766 22.7311 2.0707C24.0952 3.42827 24.7869 5.08257 24.7869 6.98762C24.7869 8.89268 24.0924 10.5504 22.7227 11.888C21.3463 13.2287 19.6634 13.9084 17.7205 13.9084ZM17.7973 2.04204C16.4439 2.04204 15.285 2.52053 14.3526 3.46411C13.4203 4.3955 12.9477 5.56208 12.9477 6.93202C12.9477 8.3685 13.4374 9.56596 14.4031 10.4913C15.365 11.4187 16.4893 11.8889 17.7452 11.8889C19.1102 11.8889 20.2779 11.411 21.2158 10.4682C22.1534 9.51389 22.6289 8.33556 22.6289 6.96632C22.6289 5.57866 22.1591 4.4004 21.2326 3.46417C20.3119 2.52041 19.1563 2.04204 17.7973 2.04204Z" fill="currentColor"></path><path d="M33.1505 13.8853H26.0156V0H28.2346V11.7917H33.1505V13.8853Z" fill="currentColor"></path><path d="M36.551 13.8853H34.3321V0H36.551V13.8853Z" fill="currentColor"></path><path d="M43.7768 13.8853H41.5578V2.09359H37.7521V0H47.674V2.09359H43.7769L43.7768 13.8853Z" fill="currentColor"></path><path d="M51.0588 13.8853H48.8405V0H51.0588V13.8853Z" fill="currentColor"></path><path d="M69.4011 13.882C67.6867 13.882 66.1232 13.2794 64.7541 12.091C63.2488 10.7741 62.4853 9.02686 62.4853 6.89755C62.4853 5.02384 63.1815 3.39205 64.5535 2.04758C65.9227 0.703803 67.5881 0.0224257 69.5042 0.0224257C71.3992 0.0224257 73.0475 0.710184 74.4024 2.06656C75.7636 3.42111 76.4536 5.07232 76.4536 6.97419C76.4536 8.88676 75.7602 10.5324 74.3933 11.8654C73.0208 13.2035 71.3417 13.882 69.4011 13.882ZM69.478 2.03807C68.1271 2.03807 66.971 2.51564 66.0412 3.45746C65.1102 4.3869 64.6379 5.55138 64.6379 6.9188C64.6379 8.35329 65.1267 9.54848 66.0908 10.4713C67.0508 11.397 68.1732 11.8663 69.4267 11.8663C70.789 11.8663 71.9542 11.3893 72.8903 10.4483C73.8264 9.49441 74.301 8.31825 74.301 6.95299C74.301 5.56807 73.8321 4.39203 72.9079 3.45746C71.9878 2.51559 70.8334 2.03807 69.478 2.03807Z" fill="currentColor"></path><path d="M2.21906 13.8859H0V0.0162442L5.42596 0.016529C7.98418 0.016529 10.0655 2.09781 10.0655 4.65615C10.0655 7.21449 7.98424 9.29577 5.42596 9.29577H2.21883L2.21906 13.8859ZM2.21883 7.26543H5.42596C6.86472 7.26543 8.03518 6.09497 8.03518 4.65615C8.03518 3.21733 6.86466 2.04687 5.42596 2.04687H2.21883V7.26543Z" fill="currentColor"></path></svg></div><div data-hs-action="reset" class="hs_quiz_button-white"><div>Start Over</div></div><div class="hs_quiz_nav_cta-wrap"><a href="https://www.bayer.com/en/" target="_blank" class="w-inline-block"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ab5ea87a76fbc84bd78ba_powered_by.webp" loading="lazy" width="107" alt="Text 'presented by' followed by Bayer's circular logo with a diagonal cross and the word 'BAYER'." class="hs_quiz_nav_powered-by"></a></div></div></div><a data-hs-element="assessment-cta" href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_header_cta w-inline-block"><div class="hs_quiz_header_cta-text">Read the report &amp; key findings</div></a></div><section class="hs_quiz_section"><div class="hs_quiz_container"><div data-hs-view="intro-wrap" class="hs_quiz_intro_wrap"><div data-hs-view="intro" class="hs_quiz_intro_card"><div class="hs_quiz_intro_title-wrap"><h1 class="hs_quiz_intro_title">Biotechnology <span class="hs_quiz_intro_title-medium">at scale</span></h1></div><div class="hs_quiz_intro_content-wrap"><div class="hs_quiz_intro_content-info-wrap"><p class="hs_quiz_intro_info-text">Biotechnology is a strategic lever of European competitiveness, industrial resilience and geopolitical influence.</p><p class="hs_quiz_intro_info-text">With the continentâ€™s strong research base, the central challenge is no longer scientific discovery alone, but whether innovation can reach consumers and translate into economic,&nbsp;environmental&nbsp;and societal value.</p><p class="hs_quiz_intro_info-text">Success will depend on whether Europe can create the enabling conditions for innovation to scale.</p></div><div class="hs_quiz_intro_cta-col"><h4 class="hs_quiz_intro_col-title">From POLITICO R&amp;A</h4><div class="hs_quiz_intro_col-content"><ul role="list" class="hs_quiz_intro_list"><li><p class="hs_quiz_intro_col-desc">Europeâ€™s biotech ecosystem in a global competitiveness race</p></li><li><p class="hs_quiz_intro_col-desc">The architecture of Europeâ€™s bioeconomy ambition</p></li><li><p class="hs_quiz_intro_col-desc">Europeâ€™s test case for preventive health innovation<br></p></li><li><p class="hs_quiz_intro_col-desc">And more<br></p></li></ul></div><div class="hs_mt-auto"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined is-intro w-inline-block"><div>Read the report</div></a></div></div><div class="hs_quiz_intro_cta-col"><h4 class="hs_quiz_intro_col-title">Can you turn biotech breakthroughs into products people use?</h4><div class="hs_quiz_intro_col-content"><p class="hs_quiz_intro_col-desc">Take on the role of an EU policymaker and see how your decisions&nbsp;impact&nbsp;the development of innovations in health care and agriculture.</p></div><div class="hs_mt-auto"><a data-hs-action="play-game" href="#" class="hs_quiz_button-navy w-inline-block"><div>Play the game</div></a></div></div></div></div><div data-hs-view="persona" class="hs_quiz_persona_card"><h4 class="hs_quiz_persona_title">Choose your persona</h4><div data-current="Agriculture policymaker" data-easing="ease" data-duration-in="300" data-duration-out="100" class="hs_quiz_persona_tab w-tabs"><div class="hs_quiz_persona_tab-menu w-tab-menu"><a data-w-tab="Agriculture policymaker" data-hs-tab-link="agriculture" class="hs_quiz_persona_tab-link w-inline-block w-tab-link w--current"><div>Agriculture policymaker</div></a><a data-w-tab="Healthcare policymaker" data-hs-tab-link="healthcare" class="hs_quiz_persona_tab-link w-inline-block w-tab-link"><div>Health care policymaker</div></a></div><div class="hs_quiz_persona_tab-content w-tab-content"><div data-w-tab="Agriculture policymaker" class="hs_quiz_persona_tab-pane w-tab-pane w--tab-active"><div class="hs_quiz_persona_tab-info"><p class="hs_quiz_persona_tab-info-text">A European seed company, <strong>Startup Y</strong>, has developed a wheat variety using New Genomic Techniques that is more resistant to heat stress. <br><br>Farmers want the seeds to help cope with increasingly extreme weather conditions. Will they reach Europe's fields in time?</p><div><p class="hs_quiz_persona_tab-info-text"><span class="hs_text-weight-medium">Your objectives</span></p><ul role="list" class="hs_quiz_intro_list mt-5"><li><p class="hs_quiz_intro_col-desc">Retain Europeâ€™s self-sufficiency</p></li><li><p class="hs_quiz_intro_col-desc">Improve food affordability</p></li><li><p class="hs_quiz_intro_col-desc">Support farmersâ€™ livelihoods</p></li><li><p class="hs_quiz_intro_col-desc">Strengthen climate resilience</p></li></ul></div></div><div class="hs_quiz_tab-button-wrap"><a data-hs-persona="agriculture" data-hs-action="start-playing" href="#" class="hs_quiz_button-green is-m-18 w-inline-block"><div>Start playing</div></a></div></div><div data-w-tab="Healthcare policymaker" class="hs_quiz_persona_tab-pane w-tab-pane"><div class="hs_quiz_persona_tab-info"><p class="hs_quiz_persona_tab-info-text">A European biotech company, <strong>Startup X</strong>, has developed a promising gene therapy that could prevent or delay Parkinson's disease in people identified as genetically at risk.<br><br>The science works. But will it reach patients?</p><div><p class="hs_quiz_persona_tab-info-text"><span class="hs_text-weight-medium">Your objectives</span></p><ul role="list" class="hs_quiz_intro_list mt-5"><li><p class="hs_quiz_intro_col-desc">Retain investment in Europe</p></li><li><p class="hs_quiz_intro_col-desc">Ensure timely patient access to new therapies</p></li><li><p class="hs_quiz_intro_col-desc">Keep medicines affordable</p></li><li><p class="hs_quiz_intro_col-desc">Maintain public trust in health systems</p></li></ul></div></div><div class="hs_quiz_tab-button-wrap"><a data-hs-persona="healthcare" data-hs-action="start-playing" href="#" class="hs_quiz_button-blue is-m-18 w-inline-block"><div>Start playing</div></a></div></div></div></div></div></div><div data-hs-view="assessment" class="hs_quiz_assessment_wrapper"><div data-hs-total-steps="5" data-hs-category="agriculture" class="hs_quiz_assessment_agriculture"><div class="hs_quiz_assessment_card"><div class="hs_quiz_assessment_header"><div class="hs_quiz_assessment_question"><span data-hs-element="qa-label" class="hs_quiz_assessment_qa-label">Question</span><span data-hs-element="qa-num" class="hs_quiz_assessment_qa-num">1</span>/<span data-hs-element="total" class="hs_quiz_assessment_total">6</span></div><div class="hs_quiz_assessment_progress-bar"><div data-hs-element="progress-fill" class="hs_quiz_assessment_progress-fill"></div></div></div><div><div data-hs-next-wrong="ag-2" data-hs-step="1" data-hs-question-id="ag-1" data-hs-next-correct="ag-2" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa13380d95464f51144066e_Icon.webp" alt="Two stylized DNA helix icons side by side, one green and one blue, with a blue not equal sign to the right." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Agriculture R&amp;D</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Agriculture is under increasing environmental pressure, and NGTs â€” which introduce targeted changes to a plantâ€™s own genome, including changes that could occur naturally or through conventional breeding â€” can develop crops that are better suited to changing conditions. How should the EU support research and development for NGTs?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="1" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Introduce dedicated funding for NGT R&amp;D projects.</div></div><div data-hs-points="1" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Reduce the administrative barriers to more R&amp;D funding.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066e15042a2d8c6d78d_exclaim.webp" alt="Orange question mark inside an irregular star shape on a pale yellow, uneven circular background with small orange marks around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">EU funding helps Startup Y accelerate research and field trials for its heat- resistant wheat. However, lengthy and complex application and approval processes cost the startup valuable time and money.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066e15042a2d8c6d78d_exclaim.webp" alt="Orange question mark inside an irregular star shape on a pale yellow, uneven circular background with small orange marks around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Reducing administrative barriers would enable Startup Y to access R&amp;D funding more easily and begin research and field trials sooner. However, simpler procedures cannot replace the need for adequate funding and scientific expertise.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="ag-3" data-hs-step="2" data-hs-question-id="ag-2" data-hs-next-correct="ag-3" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa13380afc0f54849ba38da_Icon2.webp" alt="Blue icon of a certificate with a gold seal next to a blue unlocked padlock symbol." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Agriculture IP</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">How should Europe balance patent protection for innovative NGT traits with concerns raised by farmers and breeders about access to innovation? (Breeders are organizations that develop new plant varieties.)</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Leave it to market forces â€” do not require information to be made available to farmers and breeders on patent landscape and licensing options.</div></div><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Increase transparency of registered patents and licensing platforms, and monitor the impact on innovation, seed availability and EU competitiveness.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Greater transparency can allow breeders and farmers to identify patents more easily and increase understanding of the role and functioning of licensing platforms, increasing trust and enhancing access, while preserving patent protection.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Fewer transparency obligations in theory require fewer actions from patent holders, reducing their administrative burden. However, smaller breeders and public research organizations find it harder to use patented seeds and traits, and a culture of distrust persists.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="ag-4" data-hs-step="3" data-hs-question-id="ag-3" data-hs-next-correct="ag-4" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa13380b4606992da5fdd08_Icon3.webp" alt="Green apple with a brown stem partially covered by a blue shield icon." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Agriculture regulation</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Unlike other leading global markets, Europe still regulates certain NGT plants (based on specific traits of the number of edits) with the same approach as GMOs. How should Europe regulate them moving forward?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Gradually broaden the scope of the NGT framework in line with technological progress to remove limitations on NGT plants that are scientifically not justified.</div></div><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Retain the current scope of the NGT framework.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">As the evidence develops, more NGT crops receive proportionate treatment. Startup Y attracts further investment and expands its climate-resilient pipeline.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup Yâ€™s heat-resistant wheat can proceed if its limited number of genetic changes could have occurred naturally or through conventional breeding. But more complex future varieties face tighter restrictions, limiting tools to manage climate change and other agricultural pressures.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="ag-5" data-hs-step="4" data-hs-question-id="ag-4" data-hs-next-correct="ag-5" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa13380ea1ef789891d7735_Icon4.webp" alt="Green plant with two leaves growing from soil next to a green stopwatch with speed lines indicating fast growth." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Agriculture market access</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Should NGT plants have a simplified â€˜verification procedure,â€™ ensuring a route to market comparable to conventional varieties?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">No, NGT plants should undergo more extensive checks before reaching farmersâ€™ fields.</div></div><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Yes, timely farmer access to NGT plant innovation is crucial for climate adaption in agriculture and contributes to food price stability for European consumers.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">A simplified route allows Startup Y to bring its NGT wheat to market faster and at lower cost. Conventional seed and food safeguards continue to apply.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">More extensive oversight may reassure skeptical consumers. But higher costs and longer delays push Startup Y toward more permissive markets, meaning European farmers may miss out on its innovation.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="outcome" data-hs-step="5" data-hs-question-id="ag-5" data-hs-next-correct="outcome" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa133807661b035628eb2e8_Icon5.webp" alt="Young green plant with leaves growing from soil showing roots and small nutrient particles in the soil." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Agriculture market uptake</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Public acceptance remains one of the biggest barriers to NGT adoption. Where do you focus first?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Engage key stakeholders and the wider public in a discussion around the science behind, and public-interest benefits of, NGT crops.</div></div><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Take no action to engage the public on the science or benefits behind NGT technologies â€” the transparency elements of the new regulation are sufficient.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Sharing information and an open discussion of the benefits and risks builds trust in the public, strengthens farmer demand, and helps Europe pursue food security and climate resilience without ignoring consumer choice.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Labels and databases provide information, but do not explain why Startup Yâ€™s innovation matters. Retailer caution persists, limiting uptake.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div></div></div></div><div data-hs-total-steps="6" data-hs-category="healthcare" class="hs_quiz_assessment_healthcare"><div class="hs_quiz_assessment_card"><div class="hs_quiz_assessment_header"><div class="hs_quiz_assessment_question"><span data-hs-element="qa-label" class="hs_quiz_assessment_qa-label">Question</span><span data-hs-element="qa-num" class="hs_quiz_assessment_qa-num">1</span>/<span data-hs-element="total" class="hs_quiz_assessment_total">6</span></div><div class="hs_quiz_assessment_progress-bar"><div data-hs-element="progress-fill" class="hs_quiz_assessment_progress-fill"></div></div></div><div><div data-hs-next-wrong="hc-2" data-hs-step="1" data-hs-question-id="hc-1" data-hs-next-correct="hc-2" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ea5274b94fb1f12d3ab9a_Icon1.png" alt="Blue microscopic cell illustration with darker blue internal structures and a large segmented circular feature on the right side." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care R&amp;D</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">The EU is home to a world-class biotech ecosystem; however, structural barriers have limited Europeâ€™s ability to translate research into commercial, industrial and societal value. Where do you invest first?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="1" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Strengthen biotech talent through skills and research.</div></div><div data-hs-points="1" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Reduce administrative burdens so innovation can reach patients faster.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066e15042a2d8c6d78d_exclaim.webp" alt="Orange question mark inside an irregular star shape on a pale yellow, uneven circular background with small orange marks around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Investment in skills helps Startup X recruit the specialists needed to develop and manufacture its Parkinsonâ€™s therapy. Europe strengthens its talent base and creates high-value jobs â€” but training takes time, so the companyâ€™s immediate bottlenecks remain.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066e15042a2d8c6d78d_exclaim.webp" alt="Orange question mark inside an irregular star shape on a pale yellow, uneven circular background with small orange marks around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Cutting administrative burdens enables Startup X to spend less time and money navigating complex requirements and more on bringing its therapy to patients. Commercialization accelerates, but skills shortages could still prevent the company from scaling in Europe.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="hc-3" data-hs-step="2" data-hs-question-id="hc-2" data-hs-next-correct="hc-3" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ff614949239f95186c003_Icon2.webp" alt="Stylized icon of a blue microchip inside a circular shape with abstract circuit lines and electronic components around it." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care IP</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Advanced Therapy Medicinal Products are a class of medicines based on genes, cells or tissues designed to treat, prevent or diagnose diseases. They are fundamentally different from conventional pharmaceuticals, with higher R&amp;D costs and complex manufacturing. Do ATMPs require new intellectual property rules to compensate for the increased financial risk?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Yes. Extend the exclusivity of patented products.</div></div><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">No. Keep the current framework.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Additional exclusivity reassures investors that Startup X will have longer to recoup the high costs of developing its therapy. This unlocks funding for clinical trials and European manufacturing.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X retains existing patent protections but receives no additional exclusivity. This means biosimilar competition can begin sooner, supporting lower prices now â€” but investors may be less willing to finance new biotech innovations in the future.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="hc-4" data-hs-step="3" data-hs-question-id="hc-3" data-hs-next-correct="hc-4" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ff614604aa24d12211c37_Icon3.webp" alt="Blue icon of a large laboratory flask suspended by a crane with two small human figures standing on the crane platform." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care regulation</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">How do you balance biotech companies' desire to commercialize their products with patient safety and quick access?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Maintain the current system of medicine approval.</div></div><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Create faster approval processes for ATMPs and conditions of high unmet need.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X prioritizes launching in Europe. Earlier access will reduce waiting times for European patients and pressure on health systems and carers.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">The current approval system maintains rigorous safeguards and public confidence. But Startup X sees Europe as slower in making approval and reimbursement decisions and less predictable than competing markets, and launches its therapy elsewhere first â€” leaving European patients waiting.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="hc-5" data-hs-step="4" data-hs-question-id="hc-4" data-hs-next-correct="hc-5" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ff6141f4b5e4fb2d8a35c_Icon4.webp" alt="A stylized icon of a shallow round container filled with blue liquid, sitting on a short orange and yellow stack base." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care financing</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">How do you help companies secure the financing they need to scale?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Expand existing EU innovation programs such as Horizon Europe and the European Innovation Council.</div></div><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Include biotech as a strategic priority for investment in the EU capital market reform.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X attracts the long-term private investment needed to scale in Europe. Manufacturing capacity and skilled jobs will stay in the EU, while patients will benefit from a strong domestic pipeline of new treatments.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X secures EU funding to advance clinical trials. But public programs cannot fill Europeâ€™s late-stage capital gap alone. The company may still scale elsewhere â€” taking jobs, expertise and future economic value with it.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="hc-6" data-hs-step="5" data-hs-question-id="hc-5" data-hs-next-correct="hc-6" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ff614bc5913bdd11af686_Icon5.webp" alt="Illustration of a laboratory flask pouring blue liquid onto a conveyor belt with a crane or framework structure next to it, symbolizing scientific production or processing." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care manufacturing</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">ATMPs often have a short shelf life and must be manufactured close to where clinical trials and treatment take place. How can Europe strengthen its manufacturing capacity for ATMPs?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Incentivize the creation of strategic projects for advanced therapies that integrate research, manufacturing, expertise sharing and financing to bridge the translation gap from laboratory to patient.</div></div><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Strengthen access to and coordination between existing specialized manufacturing facilities and clinical centers.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">A specialist accelerator gives Startup X access to testing, validation and small-batch production and certainty about its future in Europe. Its therapy moves closer to approval and market, while Europe builds the manufacturing skills and capacity needed to reduce reliance on medicines researched and developed elsewhere.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X gains better access to Europe's existing manufacturing network. However, existing capacity and specialized expertise remain limited, potentially creating bottlenecks as more advanced therapies reach the market.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div><div data-hs-next-wrong="outcome" data-hs-step="6" data-hs-question-id="hc-6" data-hs-next-correct="outcome" class="hs_quiz_assessment_set"><div data-hs-block="question" class="hs_quiz_assessment_q-block"><div class="hs_quiz_assessment_sector"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9ff61469c9877553fc83fb_Icon6.webp" alt="A stylized folded map with river, roads, terrain markings, and an hourglass icon above it." loading="lazy" class="hs_quiz_assessment_sector-img"><div class="hs_quiz_assessment_sector-label-wrap"><h4 class="hs_quiz_assessment_sector-label">Health care market access</h4></div></div><div class="hs_quiz_assessment_q-main"><p class="hs_quiz_assessment_text is-q">Fewer therapies get approved within the EU than in other regions, and significant gaps exist in access delays among EU countries. How do you regulate the reimbursement framework to improve access to patients?</p><div class="hs_quiz_assessment_q-options"><div data-hs-points="0" data-hs-action="select-answer" data-hs-is-correct="false" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Require companies to supply new medicines across the EU when requested, backed by incentives.</div></div><div data-hs-points="2" data-hs-action="select-answer" data-hs-is-correct="true" class="hs_quiz_assessment_q-radio"><div class="hs_quiz_assessment_q-radio-btn"></div><div class="hs_quiz_assessment_q-radio-label">Align reimbursement frameworks across EU countries and value innovation on the wider societal benefit of new therapies.</div></div></div></div></div><div data-hs-block="feedback" class="hs_quiz_assessment_a-block"><div data-hs-feedback="correct" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb0660a7f44ed565c0d8e_correct.webp" alt="Green checkmark inside a light green rounded shape with three small green star-like sparkles around it." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Aligned deadlines and recognition of the therapyâ€™s wider societal value give Startup X a predictable route across Europe. Patients gain faster, fairer access, while preventing Parkinsonâ€™s reduces pressure on carers, health systems and public finances.</p></div><div data-hs-feedback="incorrect" class="hs_quiz_assessment_a-inner-block"><div class="hs_quiz_assessment_a-img-wrap"><img width="90" src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9eb066bf831352bd6789fb_incorrect.webp" alt="Red warning triangle with exclamation mark inside, surrounded by three red X marks on a pink irregular background." loading="lazy" class="hs_quiz_assessment_a-img"></div><p class="hs_quiz_assessment_text">Startup X must negotiate with member countries and supply its new therapy when requested â€” or risk losing market protection. More countries gain access, but complex national negotiations and supply obligations increase costs and could stretch Startup Xâ€™s limited capacity.</p></div><div data-next-button="" class="hs_quiz_assessment_a-next-wrap"><button data-hs-action="next-question" class="hs_quiz_assessment_a-next-btn"><div>Next Question</div></button></div><div class="hs_quiz_assessment_a-overlay"></div></div></div></div></div></div></div><div data-hs-view="outcome" class="hs_quiz_outcome_wrapper"><div class="hs_quiz_outcome_agriculture"><div data-hs-outcome="agriculture-outcome-1" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Integrated innovation leadership</h4><p class="hs_quiz_outcome_info-text">Congratulations! Your choices have kept investment, research and seed innovation in Europe. NGT crops tailored to the challenges facing European farmers are widely available. Farmers maintain productivity despite increasing climate pressures, helping to stabilize food prices, strengthen food security, support rural prosperity, and reinforce the EU's position as a global leader in sustainable agriculture and agricultural technology.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new crop technologies to market. The choices policymakers make today will help create one of three futures for Europe.</p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="healthcare" data-hs-action="start-playing" href="#" class="hs_quiz_button-blue w-inline-block"><div>Try healthcare path</div></a></div></div></div><div data-hs-outcome="agriculture-outcome-2" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion is-2"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Constrained progress</h4><p class="hs_quiz_outcome_info-text">Not bad. Some of your decisions help support bringing NGT crops to the EU market. However, other choices create complex rules, uneven implementation across countries and uncertainty for innovators â€” discouraging investment and delaying adoption. Europe retains scientific excellence but struggles to convert research into economic, environmental and societal benefits at scale. Farmers are less able to respond to climate change than competitors in other regions, negatively affecting food prices and security.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new crop technologies to market. The choices policymakers make today will help create one of three futures for Europe.</p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="healthcare" data-hs-action="start-playing" href="#" class="hs_quiz_button-blue w-inline-block"><div>Try healthcare path</div></a></div></div></div><div data-hs-outcome="agriculture-outcome-3" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion is-3"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Fragmentation and decline</h4><p class="hs_quiz_outcome_info-text">You missed opportunities to support innovation. Your decisions lead to a complex framework, meaning investment, breeding programs and commercialization increasingly move to regions with more predictable innovation frameworks. European farmers face growing climate and competitive pressures with fewer new crop solutions available, while growers elsewhere gain access to the latest technologies first. As innovation and production shift abroad, European farmers fall further behind their global counterparts, weakening food security, rural competitiveness and long-term strategic resilience.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new crop technologies to market. The choices policymakers make today will help create one of three futures for Europe.</p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="healthcare" data-hs-action="start-playing" href="#" class="hs_quiz_button-blue w-inline-block"><div>Try healthcare path</div></a></div></div></div></div><div class="hs_quiz_outcome_healthcare"><div data-hs-outcome="healthcare-outcome-1" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Integrated innovation leadership</h4><p class="hs_quiz_outcome_info-text">Congratulations! You supported and expanded biotech innovation by reducing regulatory complexity, improving access to financing and backing specialized biotechnology development accelerators and strategic projects. Thanks to your decisions, patients in Europe get faster and fairer access to innovative treatments. Skilled jobs stay in the EU, and Europeâ€™s health-system resilience is strong.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new therapies to market. The choices policymakers make today will help create one of three futures for Europe. <br></p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="agriculture" data-hs-action="start-playing" href="#" class="hs_quiz_button-green w-inline-block"><div>Try agriculture path</div></a></div></div></div><div data-hs-outcome="healthcare-outcome-2" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion is-2"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Constrained progress</h4><p class="hs_quiz_outcome_info-text">You made some good decisions, however some of your choices have enabled the continuation of fragmented financing, manufacturing and reimbursement in Europe. This means that biotech therapies reach some patients late â€” and others not at all â€” and Europe continues to lag behind other regions in both access and industrial capacity.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new therapies to market. The choices policymakers make today will help create one of three futures for Europe.</p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="agriculture" data-hs-action="start-playing" href="#" class="hs_quiz_button-green w-inline-block"><div>Try agriculture path</div></a></div></div></div><div data-hs-outcome="healthcare-outcome-3" class="hs_quiz_outcome_card"><h4 class="hs_quiz_outcome_title">Your outcome</h4><div class="hs_quiz_outcome_content"><div class="hs_quiz_outcome_fixed-wrap"><div class="hs_quiz_outcome_accordion is-3"><div class="hs_quiz_outcome_accordion_trigger"><div class="hs_quiz_outcome_accordion_trigger-wrap"><h4 class="hs_quiz_outcome_accordion_title">Fragmentation and decline</h4><p class="hs_quiz_outcome_info-text">You missed the opportunity to strengthen the biotech ecosystem in Europe. Without making choices that provide EU-level support and harmonized regulatory frameworks, biotech companies move investment and production abroad. This leaves European patients waiting longer and paying more for a therapy developed elsewhere and Europe without a strong industrial footprint supporting economic growth.</p><p class="hs_quiz_outcome_info-text">Now you understand the complex challenges companies face when bringing new therapies to market. The choices policymakers make today will help create one of three futures for Europe.<br></p></div><div class="hs_quiz_outcome_accordion_icon"><div class="hs_quiz_outcome_accordion_line"></div><div class="hs_quiz_outcome_accordion_line is-2"></div></div></div></div></div><div class="hs_quiz_outcome_button-wrap"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_button-outlined w-inline-block"><div>View full report</div></a><a data-hs-persona="agriculture" data-hs-action="start-playing" href="#" class="hs_quiz_button-green w-inline-block"><div>Try agriculture path</div></a></div></div></div></div></div></div><div class="hs_quiz_section_bg"><div data-hs-bg="intro" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day.webp" loading="eager" width="1920" sizes="100vw" alt="Illustration of rolling green fields with bushes in the foreground, a small house with two sheep on a hill, and a city skyline with tall buildings under a blue sky with clouds." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6a9abb76a1a4f50a9cd084fa_Agriculture_mid%20day.webp 3840w" class="hs_quiz_section_bg_img is-1"><div class="hs_quiz_section_bg_img-overlay is-1"></div></div><div data-hs-bg="persona" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg.webp" loading="lazy" width="1920" sizes="100vw" alt="Split scene with a person wearing a green cap and vest using a tablet while standing in a green farm field with flowers on the left, and a scientist in a white lab coat reviewing a clipboard inside a lab with a microscope, laptop, and plant on the right, with a cityscape visible in the background." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2ad0e590811c1de495568_persona_bg.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-2"></div></div><div data-hs-bg="hc-1" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning.webp" loading="lazy" width="1920" sizes="100vw" alt="Bright laboratory room with a large window showing a cityscape and greenery outside, lab equipment including a microscope, test tubes, a laptop displaying a pie chart, a beaker on a heating plate, a distillation setup with a pressure gauge, potted plants, and shelves with bottles." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa28752bb62f87ec3477cbd_Laboratory_Morning.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-4"></div></div><div data-hs-bg="hc-2" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202.webp" loading="lazy" width="1920" sizes="100vw" alt="A modern bright laboratory workspace with a microscope, laptop showing a pie chart, test tubes, beaker, scientific equipment with a gauge and burner, potted plants on the counter and shelf, soap dispenser near a sink, and a window showing a green park and cityscape outside." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="hc-3" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3.webp" loading="lazy" width="1920" sizes="100vw" alt="Bright laboratory workspace with scientific equipment including a microscope, test tubes, a laptop displaying graphs, and a distillation apparatus with a Bunsen burner, under a window showing a green park and a city skyline with buildings and a church tower in warm sunlight." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2b00e287f8a07fdeb28c5_Laboratory_Night_3.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="hc-4" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4.webp" loading="lazy" width="1920" sizes="100vw" alt="Bright laboratory countertop with scientific equipment including a microscope, laptop displaying data, test tubes in a rack, beakers with blue liquid on a hot plate, and a distillation setup with a gauge above a lit flame, all under a window showing a sunny cityscape and green plants indoors." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffb5bbb3d12aa224ab_Laboratory_Morning_4.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="hc-5" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a laboratory workspace featuring scientific equipment such as a microscope, test tubes in a rack, a digital laptop displaying a pie chart, a heating element with a beaker, a distillation setup with a gauge, and various bottles on shelves, with a window showing a cityscape and greenery outside." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="hc-6" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2.webp" loading="lazy" width="1920" sizes="100vw" alt="Nighttime laboratory scene with a microscope, an open laptop displaying a pie chart, test tubes, laboratory glassware with a flame underneath, and various bottles and plants, illuminated by a window showing a dark cityscape with lit windows and a crescent moon." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="ag-1" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a rural landscape with rolling yellow fields, green bushes with white flowers in the foreground, and a small house with a red roof on a hill, against a city skyline with varied buildings under a partly cloudy sky." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="ag-2" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a green countryside with rolling fields, small house, trees, flowers, and sheep, set against a backdrop of a city skyline with tall buildings and a church steeple under a partly cloudy sky." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="ag-3" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a green countryside with rolling hills, bushes, white flowers, a small house with trees, and sheep grazing, with a city skyline of blue and white buildings and a church spire in the background under a bright blue sky with scattered clouds." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7231ce9aa3d30402_desktop-4.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="ag-4" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a peaceful rural landscape with rolling yellow fields, green bushes with white flowers in the foreground, a small house with trees and sheep on the right, and a town with buildings and a church spire in the distance under a sky with soft clouds." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcc22bf4c344a2a0bd_desktop-3.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-3"></div></div><div data-hs-bg="ag-5" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2.webp" loading="lazy" width="1920" sizes="100vw" alt="Nighttime cityscape illustration with a crescent moon, dark blue sky, glowing yellow-lit windows in buildings, and green rolling hills with scattered flowers in the foreground." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-5"></div></div><div data-hs-bg="healthcare-outcome-1" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202.webp" loading="lazy" width="1920" sizes="100vw" alt="A modern bright laboratory workspace with a microscope, laptop showing a pie chart, test tubes, beaker, scientific equipment with a gauge and burner, potted plants on the counter and shelf, soap dispenser near a sink, and a window showing a green park and cityscape outside." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2af680bd93e960f2b91e1_Laboratory_Day%202.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div><div data-hs-bg="healthcare-outcome-2" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a laboratory workspace featuring scientific equipment such as a microscope, test tubes in a rack, a digital laptop displaying a pie chart, a heating element with a beaker, a distillation setup with a gauge, and various bottles on shelves, with a window showing a cityscape and greenery outside." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffcdfe6841f96d3aa6_Laboratory_Day_3.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div><div data-hs-bg="healthcare-outcome-3" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2.webp" loading="lazy" width="1920" sizes="100vw" alt="Nighttime laboratory scene with a microscope, an open laptop displaying a pie chart, test tubes, laboratory glassware with a flame underneath, and various bottles and plants, illuminated by a window showing a dark cityscape with lit windows and a crescent moon." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2aeffd48263f65287e252_Laboratory_Night_2.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div><div data-hs-bg="agriculture-outcome-1" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a green countryside with rolling fields, small house, trees, flowers, and sheep, set against a backdrop of a city skyline with tall buildings and a church steeple under a partly cloudy sky." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebc7cf3a262ff6e830e_desktop.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div><div data-hs-bg="agriculture-outcome-2" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1.webp" loading="lazy" width="1920" sizes="100vw" alt="Illustration of a rural landscape with rolling yellow fields, green bushes with white flowers in the foreground, and a small house with a red roof on a hill, against a city skyline with varied buildings under a partly cloudy sky." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebeafcbf245d441b4d3_desktop-1.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div><div data-hs-bg="agriculture-outcome-3" class="hs_quiz_section_bg_img-wrap"><img src="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2.webp" loading="lazy" width="1920" sizes="100vw" alt="Nighttime cityscape illustration with a crescent moon, dark blue sky, glowing yellow-lit windows in buildings, and green rolling hills with scattered flowers in the foreground." srcset="https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-500.webp 500w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-800.webp 800w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-1080.webp 1080w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-1600.webp 1600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-2000.webp 2000w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-2600.webp 2600w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2-p-3200.webp 3200w, https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/6aa2cebcf71a3638f407bbb1_desktop-2.webp 3840w" class="hs_quiz_section_bg_img"><div class="hs_quiz_section_bg_img-overlay is-outcome"></div></div></div><div data-hs-element="assessment-cta" class="hs_quiz_mobile-cta"><a href="https://www.politico.eu/research-and-analysis/biotechnology-at-scale-europes-next-competitiveness-test-report/" target="_blank" class="hs_quiz_header_cta is-mobile w-inline-block"><div class="hs_quiz_header_cta-text">Read the report &amp; key findings</div></a></div></section><div class="hs_hide w-embed w-script"></div></div>`;
    currentScript.after(container);

    var restoreListeners = patchReadyListeners();

    var externalScripts = [
  "https://d3e54v103j8qbb.cloudfront.net/js/jquery-3.5.1.min.dc5e7f18c8.js?site=6a9a9a6d2876b5a5a344308c",
  "https://cdn.prod.website-files.com/6a9a9a6d2876b5a5a344308c/js/webflow.7fdf50bc.e09d923bc231ffd4.js"
];

    var inlineScripts = [
    `!function(o,c){var n=c.documentElement,t=" w-mod-";n.className+=t+"js",("ontouchstart"in o||o.DocumentTouch&&c instanceof DocumentTouch)&&(n.className+=t+"touch")}(window,document);`,
    `// Append Quiz Nav to Header el
  const header = document.querySelector('header');
  const quizNav = document.querySelector('.hs_quiz_header');

  if (header && quizNav) {
    quizNav.style.gridColumn = '1 / -1';
    quizNav.style.position = 'relative';
    quizNav.style.width = '100vw';
    quizNav.style.left = '50%';
    quizNav.style.right = '50%';
    quizNav.style.marginLeft = '-50vw';
    quizNav.style.marginRight = '-50vw';
    quizNav.style.boxSizing = 'border-box';

    header.append(quizNav);
  }`,
    `/**
   * Bayer Quiz Engine (Background Fade Only - Vev Safe, Points Based, CTA Toggle, Outcome Routing)
   */
  const QuizEngine = {
    state: {
      currentView: 'intro',
      persona: null,
      currentStep: 0,
      totalSteps: 0,
      currentQuestionId: null,
      nextTargetId: null,
      history: [],
    },

    init() {
      this.injectCSS();
      this.bindEvents();
      sessionStorage.removeItem('hsQuizHistory');

      // [VEV FIX]: Force the main tag to expand and remove padding
      const mainTag = document.querySelector('main');
      if (mainTag) {
        mainTag.style.maxWidth = 'none';
        mainTag.style.paddingLeft = '0';
        mainTag.style.paddingRight = '0';
      }

      // Hide Assessment CTAs and Reset buttons on initial load
      document.querySelectorAll('[data-hs-element="assessment-cta"]').forEach((cta) => {
        cta.style.display = 'none';
      });
      document.querySelectorAll('[data-hs-action="reset"]').forEach((btn) => {
        btn.style.display = 'none';
      });

      // Instantly show the intro background without any fading on load
      const introBg = document.querySelector('[data-hs-bg="intro"]');
      if (introBg) {
        introBg.style.display = 'block';
        introBg.style.opacity = '1';
      }
    },

    injectCSS() {
      if (document.getElementById('hs-quiz-styles')) return;
      const style = document.createElement('style');
      style.id = 'hs-quiz-styles';
      style.innerHTML = \`
      [data-hs-bg] {
        transition: opacity 0.4s ease;
      }
    \`;
      document.head.appendChild(style);
    },

    updateBackground(bgId) {
      const activeBg = document.querySelector(\`[data-hs-bg="\${bgId}"]\`);
      if (activeBg) {
        document.querySelectorAll('[data-hs-bg]').forEach((el) => {
          if (el !== activeBg) {
            el.style.opacity = '0';
            setTimeout(() => {
              el.style.display = 'none';
            }, 400);
          }
        });
        activeBg.style.display = 'block';
        setTimeout(() => {
          activeBg.style.opacity = '1';
        }, 10);
      }
    },

    setView(viewName) {
      this.state.currentView = viewName;

      document.querySelectorAll('[data-hs-element="assessment-cta"]').forEach((cta) => {
        cta.style.display = viewName === 'assessment' ? '' : 'none';
      });

      document.querySelectorAll('[data-hs-action="reset"]').forEach((btn) => {
        btn.style.display = viewName === 'intro' ? 'none' : '';
      });

      const introCard = document.querySelector('[data-hs-view="intro"]');
      const introWrap = document.querySelector('[data-hs-view="intro-wrap"]');
      const personaCard = document.querySelector('[data-hs-view="persona"]');
      const assessmentWrap = document.querySelector('[data-hs-view="assessment"]');
      const outcomeWrap = document.querySelector('[data-hs-view="outcome"]');

      if (viewName === 'persona') {
        if (introWrap) introWrap.style.display = 'flex';
        if (personaCard) personaCard.style.display = 'block';

        if (introCard) introCard.style.display = 'none';
        if (assessmentWrap) assessmentWrap.style.display = 'none';
        if (outcomeWrap) outcomeWrap.style.display = 'none';

        this.updateBackground('persona');
      } else if (viewName === 'assessment') {
        if (introWrap) introWrap.style.display = 'none';
        if (outcomeWrap) outcomeWrap.style.display = 'none';
        if (assessmentWrap) assessmentWrap.style.display = 'flex';
      } else if (viewName === 'outcome') {
        if (introWrap) introWrap.style.display = 'none';
        if (assessmentWrap) assessmentWrap.style.display = 'none';
        if (outcomeWrap) outcomeWrap.style.display = 'flex';
      } else if (viewName === 'intro') {
        if (introWrap) introWrap.style.display = 'flex';
        if (introCard) introCard.style.display = 'block';

        if (personaCard) personaCard.style.display = 'none';
        if (assessmentWrap) assessmentWrap.style.display = 'none';
        if (outcomeWrap) outcomeWrap.style.display = 'none';

        this.updateBackground('intro');
      }
    },

    setPersona(personaName) {
      this.state.persona = personaName;

      document.querySelectorAll('[data-hs-category]').forEach((el) => (el.style.display = 'none'));

      const categoryWrap = document.querySelector(\`[data-hs-category="\${personaName}"]\`);
      if (categoryWrap) {
        categoryWrap.style.display = 'block';
        this.state.totalSteps = parseInt(categoryWrap.getAttribute('data-hs-total-steps') || 6);

        const totalEl = categoryWrap.querySelector('[data-hs-element="total"]');
        if (totalEl) totalEl.textContent = this.state.totalSteps;

        const firstQuestion = categoryWrap.querySelector('.hs_quiz_assessment_set');
        if (firstQuestion) {
          this.goToQuestion(firstQuestion.getAttribute('data-hs-question-id'));
        }
      }
    },

    goToQuestion(questionId) {
      document.querySelectorAll('.hs_quiz_assessment_set').forEach((el) => {
        el.style.display = 'none';
        const qBlock = el.querySelector('[data-hs-block="question"]');
        const aBlock = el.querySelector('[data-hs-block="feedback"]');
        if (qBlock) qBlock.style.display = 'flex';
        if (aBlock) aBlock.style.display = 'none';
      });

      const nextSet = document.querySelector(\`[data-hs-question-id="\${questionId}"]\`);

      if (nextSet) {
        this.state.currentQuestionId = questionId;
        this.state.currentStep = parseInt(nextSet.getAttribute('data-hs-step') || 0);

        nextSet.style.display = 'flex';

        this.updateHeader('Question');
        this.updateBackground(questionId);
      } else {
        this.handleOutcome();
      }
    },

    submitAnswer(labelEl) {
      const questionSet = labelEl.closest('.hs_quiz_assessment_set');
      if (!questionSet) return;

      if (questionSet.classList.contains('is-answered')) return;
      questionSet.classList.add('is-answered');

      const isCorrect = labelEl.getAttribute('data-hs-is-correct') === 'true';
      const points = parseInt(labelEl.getAttribute('data-hs-points') || '0', 10);

      let nextTarget = isCorrect ? questionSet.getAttribute('data-hs-next-correct') : questionSet.getAttribute('data-hs-next-wrong');

      this.state.nextTargetId = nextTarget;

      this.state.history.push({
        questionId: this.state.currentQuestionId,
        isCorrect: isCorrect,
        points: points,
      });

      sessionStorage.setItem('hsQuizHistory', JSON.stringify(this.state.history));

      const qBlock = questionSet.querySelector('[data-hs-block="question"]');
      const aBlock = questionSet.querySelector('[data-hs-block="feedback"]');
      if (qBlock) qBlock.style.display = 'none';
      if (aBlock) aBlock.style.display = 'flex';

      const correctMsg = questionSet.querySelector('[data-hs-feedback="correct"]');
      const wrongMsg = questionSet.querySelector('[data-hs-feedback="incorrect"]');

      if (isCorrect) {
        if (correctMsg) correctMsg.style.display = 'flex';
        if (wrongMsg) wrongMsg.style.display = 'none';
      } else {
        if (correctMsg) correctMsg.style.display = 'none';
        if (wrongMsg) wrongMsg.style.display = 'flex';
      }

      // If this is the final question, update the Next button text
      if (this.state.currentStep === this.state.totalSteps) {
        const nextBtns = questionSet.querySelectorAll('[data-hs-action="next-question"]');
        nextBtns.forEach((btn) => {
          if (btn.firstElementChild) {
            btn.firstElementChild.textContent = 'See the outcome';
          } else {
            btn.textContent = 'See the outcome';
          }
        });
      }

      this.updateHeader('Answer', true);
    },

    updateHeader(labelText, isAnswerPhase = false) {
      const categoryWrap = document.querySelector(\`[data-hs-category="\${this.state.persona}"]\`);
      if (!categoryWrap) return;

      const labelEl = categoryWrap.querySelector('[data-hs-element="qa-label"]');
      if (labelEl) labelEl.textContent = labelText;

      if (!isAnswerPhase && this.state.currentStep > 0) {
        const numEl = categoryWrap.querySelector('[data-hs-element="qa-num"]');
        if (numEl) numEl.textContent = this.state.currentStep;

        const progressFill = categoryWrap.querySelector('[data-hs-element="progress-fill"]');
        if (progressFill && this.state.totalSteps > 0) {
          const percentage = (this.state.currentStep / this.state.totalSteps) * 100;
          progressFill.style.width = percentage + '%';
        }
      }
    },

    handleOutcome() {
      const history = this.state.history;
      const totalPoints = history.reduce((sum, item) => sum + item.points, 0);

      let outcomeType = '';
      if (totalPoints >= 9) {
        outcomeType = 'outcome-1';
      } else if (totalPoints >= 5) {
        outcomeType = 'outcome-2';
      } else {
        outcomeType = 'outcome-3';
      }

      this.setView('outcome');

      document.querySelectorAll('[data-hs-outcome]').forEach((el) => (el.style.display = 'none'));

      const specificOutcome = document.querySelector(\`[data-hs-outcome="\${this.state.persona}-\${outcomeType}"]\`);
      if (specificOutcome) specificOutcome.style.display = 'block';

      this.updateBackground(\`\${this.state.persona}-\${outcomeType}\`);
    },

    /**
     * Resilient Tab Switcher
     * Polls the DOM layout tree to ensure Webflow Tabs are safely clickable after routing.
     */
    clickWebflowTab(persona, attempt = 1) {
      if (!persona) return;

      const cleanPersona = persona.trim();
      const allRegisteredTabs = Array.from(document.querySelectorAll('[data-hs-tab-link]')).map((el) => ({
        attrValue: el.getAttribute('data-hs-tab-link'),
        element: el,
      }));

      // 1. Attempt exact attribute match
      let marker = document.querySelector(\`[data-hs-tab-link="\${cleanPersona}"]\`);

      // 2. Case-insensitive fallback
      if (!marker) {
        marker = allRegisteredTabs.find((t) => (t.attrValue || '').trim().toLowerCase() === cleanPersona.toLowerCase())?.element;
      }

      // 3. Last-resort fallback to first available tab
      if (!marker && allRegisteredTabs.length > 0) {
        marker = allRegisteredTabs[0].element;
      }

      if (!marker) return;

      // 4. Resolve the actual Webflow clickable tab link (.w-tab-link)
      const tabLink = marker.classList.contains('w-tab-link') ? marker : marker.closest('.w-tab-link') || marker.querySelector('.w-tab-link') || marker;

      // 5. Visibility Polling (Crucial for Vev Embeds)
      const isVisibleInLayout = tabLink.offsetParent !== null;

      // Wait and retry (up to 5 frames / 300ms) until the browser layout paints the element
      if (!isVisibleInLayout && attempt < 5) {
        setTimeout(() => {
          this.clickWebflowTab(persona, attempt + 1);
        }, 60);
        return;
      }

      // 6. Execute Native & jQuery Clicks
      tabLink.click();

      if (window.jQuery) {
        window.jQuery(tabLink).triggerHandler('tap');
        window.jQuery(tabLink).trigger('click');
      }
    },

    bindEvents() {
      document.body.addEventListener('click', (e) => {
        const playBtn = e.target.closest('[data-hs-action="play-game"]');
        if (playBtn) {
          e.preventDefault();
          this.setView('persona');
        }

        const startBtn = e.target.closest('[data-hs-action="start-playing"]');
        if (startBtn) {
          e.preventDefault();
          const persona = startBtn.getAttribute('data-hs-persona');

          // OUTCOME ROUTING: Return to persona view and trigger tab logic
          if (startBtn.closest('[data-hs-view="outcome"]')) {
            this.setView('persona');

            if (persona) {
              // A short 50ms delay lets the browser set display:flex on introWrap
              setTimeout(() => {
                this.clickWebflowTab(persona);
              }, 50);
            }
            return;
          }

          // NORMAL START ROUTING: Start the quiz
          this.state.history = [];
          sessionStorage.removeItem('hsQuizHistory');
          document.querySelectorAll('.hs_quiz_assessment_set.is-answered').forEach((el) => {
            el.classList.remove('is-answered');
          });

          this.setView('assessment');
          this.setPersona(persona);
        }

        const answerLabel = e.target.closest('[data-hs-action="select-answer"]');
        if (answerLabel) {
          this.submitAnswer(answerLabel);
        }

        const nextBtn = e.target.closest('[data-hs-action="next-question"]');
        if (nextBtn) {
          e.preventDefault();
          this.goToQuestion(this.state.nextTargetId);
        }

        const resetBtn = e.target.closest('[data-hs-action="reset"], [data-reset]');
        if (resetBtn) {
          e.preventDefault();

          // 1. Wipe out history and state
          this.state.history = [];
          this.state.currentStep = 0;
          this.state.persona = null;
          this.state.currentQuestionId = null;
          this.state.nextTargetId = null;
          sessionStorage.removeItem('hsQuizHistory');

          // 2. Reset the visual states of all questions
          document.querySelectorAll('.hs_quiz_assessment_set').forEach((el) => {
            el.classList.remove('is-answered');

            const qBlock = el.querySelector('[data-hs-block="question"]');
            const aBlock = el.querySelector('[data-hs-block="feedback"]');
            if (qBlock) qBlock.style.display = 'flex';
            if (aBlock) aBlock.style.display = 'none';

            // Reset the "See the outcome" button back to "Next Question"
            const nextBtns = el.querySelectorAll('[data-hs-action="next-question"]');
            nextBtns.forEach((btn) => {
              if (btn.firstElementChild) {
                btn.firstElementChild.textContent = 'Next Question';
              } else {
                btn.textContent = 'Next Question';
              }
            });
          });

          // 3. Route back to intro
          this.setView('intro');
        }
      });
    },
  };

  document.addEventListener('DOMContentLoaded', () => QuizEngine.init());`
    ];

    loadScriptsInOrder(externalScripts).then(function() {
      setTimeout(function() {
        runInlineScripts(inlineScripts);
        restoreListeners();
        container.dispatchEvent(new CustomEvent('hlabs:loaded', { bubbles: true }));
        console.log('[HlabsEmbed] Loaded â€” ' + externalScripts.length + ' scripts, direct DOM injection');
      }, 100);
    });
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
})();
