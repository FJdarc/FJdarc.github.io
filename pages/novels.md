---
layout: mypost
title: 小说
---

<style>
  [data-role="inline-controls"][data-sticky="true"] {
    position: sticky;
    top: 0;
    z-index: 10;
    background: transparent;
    padding: 8px 0;
  }

  [data-role="chapter-dropdown"] {
    position: relative;
    display: inline-block;
    margin-left: 12px;
  }

  [data-role="chapter-dropdown-toggle"] {
    width: 100%;
    background: transparent;
    border: 1px solid currentColor;
    color: #333333;
    white-space: nowrap;
    box-sizing: border-box;
  }

  [data-role="inline-controls"] button {
    color: #333333;
    border-color: #333333;
  }

  [data-role="chapter-dropdown-menu"] {
    position: absolute;
    top: calc(100% + 4px);
    left: 0;
    width: 100%;
    min-width: 100%;
    max-height: 240px;
    overflow-y: auto;
    border: 1px solid currentColor;
    background: rgba(255, 255, 255, 0.12);
    z-index: 20;
    box-sizing: border-box;
  }

  html.dark [data-role="chapter-dropdown-menu"],
  body.dark [data-role="chapter-dropdown-menu"] {
    background: rgba(255, 255, 255, 0.12);
    color: #c9d1d9;
  }

  [data-role="chapter-dropdown-menu"][hidden] {
    display: none;
  }

  [data-role="chapter-option"] {
    display: block;
    width: 100%;
    text-align: left;
    background: transparent;
    border: 0;
    color: #333333;
    padding: 6px 10px;
    box-sizing: border-box;
    white-space: nowrap;
    transition: background-color 160ms ease, color 160ms ease, box-shadow 160ms ease;
  }

  [data-role="chapter-option"]:hover,
  [data-role="chapter-option"]:focus,
  [data-role="chapter-option"]:focus-visible {
    background: rgba(51, 51, 51, 0.12);
    box-shadow: inset 3px 0 0 currentColor;
    outline: none;
  }

  [data-role="chapter-option"][data-selected="true"] {
    background: rgba(255, 255, 255, 0.12);
    box-shadow: inset 3px 0 0 currentColor;
  }

  [data-role="chapter-option"][data-selected="true"]:hover,
  [data-role="chapter-option"][data-selected="true"]:focus,
  [data-role="chapter-option"][data-selected="true"]:focus-visible {
    background: rgba(51, 51, 51, 0.12);
  }

  [data-role="novels-list"],
  [data-role="novels-list"] a,
  [data-role="novels-list"] span {
    color: #333333;
  }

  [data-role="novels-list"] a.hover-underline {
    border-bottom: 1px solid #333333;
    text-decoration: none;
  }

  [data-role="novels-list"] a.hover-underline::after {
    display: none;
  }

  html.dark [data-role="novels-list"],
  html.dark [data-role="novels-list"] a,
  html.dark [data-role="novels-list"] span {
    color: #c9d1d9;
  }

  html.dark [data-role="novels-list"] a.hover-underline {
    border-bottom-color: #c9d1d9;
  }

  html.dark [data-role="chapter-dropdown-toggle"],
  body.dark [data-role="chapter-dropdown-toggle"],
  html.dark [data-role="chapter-option"],
  body.dark [data-role="chapter-option"],
  html.dark [data-role="inline-controls"] button,
  body.dark [data-role="inline-controls"] button {
    color: #c9d1d9;
    border-color: #c9d1d9;
  }

  html.dark [data-role="chapter-option"]:hover,
  html.dark [data-role="chapter-option"]:focus,
  html.dark [data-role="chapter-option"]:focus-visible,
  body.dark [data-role="chapter-option"]:hover,
  body.dark [data-role="chapter-option"]:focus,
  body.dark [data-role="chapter-option"]:focus-visible {
    background: rgba(255, 255, 255, 0.12);
    box-shadow: inset 3px 0 0 currentColor;
    outline: none;
  }

  html.dark [data-role="chapter-option"][data-selected="true"],
  body.dark [data-role="chapter-option"][data-selected="true"] {
    background: rgba(255, 255, 255, 0.12);
    box-shadow: inset 3px 0 0 currentColor;
  }

  html.dark [data-role="chapter-option"][data-selected="true"]:hover,
  html.dark [data-role="chapter-option"][data-selected="true"]:focus,
  html.dark [data-role="chapter-option"][data-selected="true"]:focus-visible,
  body.dark [data-role="chapter-option"][data-selected="true"]:hover,
  body.dark [data-role="chapter-option"][data-selected="true"]:focus,
  body.dark [data-role="chapter-option"][data-selected="true"]:focus-visible {
    background: rgba(255, 255, 255, 0.12);
  }
</style>

<div data-role="novels-app">
  <section data-stage="novels">
    <ul data-role="novels-list">
      {% for novel in site.novels %}
        <li
          data-role="novel-card"
          data-novel-name="{{ novel.name | escape }}"
          data-novel-author="{{ novel.author | escape }}"
          data-chapters='{{ novel.chapter | jsonify | escape }}'
        >
          <p>
            <a href="#" data-action="open-novel" class="hover-underline">{{ novel.name }}</a>
            <span style="float: right;">{{ novel.author }}</span>
          </p>
          <div data-role="inline-controls" hidden></div>
          <div data-role="inline-reader-status"></div>
          <div data-role="inline-reader" hidden>
            <iframe data-role="chapter-frame" loading="lazy" scrolling="no" hidden></iframe>
          </div>
        </li>
      {% endfor %}
    </ul>
  </section>
</div>

<script>
  (function () {
    var app = document.querySelector('[data-role="novels-app"]');

    if (!app) {
      return;
    }

    var novelCards = app.querySelectorAll('[data-role="novel-card"]');

    var state = {
      currentNovelCard: null,
      currentNovel: null,
      currentChapter: null,
      currentItems: [],
      currentIndex: -1,
      chapterRequestId: 0
    };

    function escapeHtml(value) {
      return String(value == null ? '' : value).replace(/[&<>"']/g, function (char) {
        return {
          '&': '&amp;',
          '<': '&lt;',
          '>': '&gt;',
          '"': '&quot;',
          "'": '&#39;'
        }[char];
      });
    }

    function forEachNode(list, callback) {
      Array.prototype.forEach.call(list || [], callback);
    }

    function getFrameDocument(frame) {
      try {
        return frame && (frame.contentDocument || (frame.contentWindow && frame.contentWindow.document));
      } catch (error) {
        return null;
      }
    }

    function resetFrame(frame) {
      if (!frame) {
        return;
      }

      frame.hidden = true;
      frame.removeAttribute('src');
      frame.style.height = '0px';
      frame.style.width = '100%';
      frame.style.maxWidth = '100%';
      frame.style.display = 'block';
      frame.style.border = '0';
      frame.style.overflowX = 'auto';
      frame.style.overflowY = 'hidden';
      frame.style.margin = '0';
      frame.style.padding = '0';

      if (frame.parentNode) {
        frame.parentNode.hidden = true;
      }
    }

    function closeDropdown(card) {
      var menu = card && card.querySelector('[data-role="chapter-dropdown-menu"]');

      if (menu) {
        menu.hidden = true;
      }
    }

    function updateChapterSelection(card, chapter) {
      var toggle = card && card.querySelector('[data-role="chapter-dropdown-toggle"]');
      var options = card ? card.querySelectorAll('[data-role="chapter-option"]') : [];
      var matchedOption = null;

      forEachNode(options, function (option) {
        var selected = option.getAttribute('data-value') === String(chapter);

        if (selected) {
          matchedOption = option;
          option.setAttribute('data-selected', 'true');
        } else {
          option.removeAttribute('data-selected');
        }
      });

      if (toggle && matchedOption) {
        toggle.textContent = matchedOption.textContent;
      }
    }

    function getElementChildrenByLocalName(parent, localName) {
      var result = [];

      if (!parent || !parent.children) {
        return result;
      }

      Array.prototype.forEach.call(parent.children, function (node) {
        if (node.nodeType === 1 && node.localName === localName) {
          result.push(node);
        }
      });

      return result;
    }

    function getFirstElementChildByLocalName(parent, localName) {
      var children = getElementChildrenByLocalName(parent, localName);
      return children.length ? children[0] : null;
    }

    function getTextContentByPath(parent, path) {
      var current = parent;
      var i;

      for (i = 0; i < path.length; i += 1) {
        current = getFirstElementChildByLocalName(current, path[i]);

        if (!current) {
          return '';
        }
      }

      return (current.textContent || '').trim();
    }

    function parseNavPoint(navPoint) {
      var contentNode = getFirstElementChildByLocalName(navPoint, 'content');
      var childNavPoints = getElementChildrenByLocalName(navPoint, 'navPoint');
      var children = [];

      childNavPoints.forEach(function (child) {
        children.push(parseNavPoint(child));
      });

      return {
        text: getTextContentByPath(navPoint, ['navLabel', 'text']),
        src: contentNode ? contentNode.getAttribute('src') || '' : '',
        children: children
      };
    }

    function flattenNavItems(items, result) {
      items.forEach(function (item) {
        if (item.src) {
          result.push({
            text: item.text || item.src.split('/').pop(),
            src: item.src
          });
        }

        if (item.children && item.children.length) {
          flattenNavItems(item.children, result);
        }
      });
    }

    function buildManifestMap(opf) {
      var manifest = opf ? opf.getElementsByTagName('manifest')[0] : null;
      var items = manifest ? manifest.getElementsByTagName('item') : [];
      var map = {};

      Array.prototype.forEach.call(items, function (item) {
        var id = item.getAttribute('id');
        var href = item.getAttribute('href');
        var mediaType = item.getAttribute('media-type') || '';

        if (id) {
          map[id] = {
            id: id,
            href: href || '',
            mediaType: mediaType
          };
        }
      });

      return map;
    }

    function buildSpineItems(opf, manifestMap) {
      var spine = opf ? opf.getElementsByTagName('spine')[0] : null;
      var itemrefs = spine ? spine.getElementsByTagName('itemref') : [];
      var items = [];

      Array.prototype.forEach.call(itemrefs, function (itemref) {
        var idref = itemref.getAttribute('idref');
        var manifestItem = idref ? manifestMap[idref] : null;

        if (manifestItem && /application\/xhtml\+xml/i.test(manifestItem.mediaType)) {
          items.push({
            text: '',
            src: manifestItem.href
          });
        }
      });

      return items;
    }

    function mergeTocWithSpine(navItems, spineItems) {
      var flatNavItems = [];
      var navMap = {};
      var merged = [];

      flattenNavItems(navItems, flatNavItems);

      flatNavItems.forEach(function (item) {
        if (item.src) {
          navMap[item.src] = item;
        }
      });

      spineItems.forEach(function (item) {
        if (!item.src) {
          return;
        }

        if (navMap[item.src]) {
          merged.push({
            text: navMap[item.src].text || item.src.split('/').pop(),
            src: item.src
          });
        } else {
          merged.push({
            text: item.src.split('/').pop(),
            src: item.src
          });
        }
      });

      return merged;
    }

    function parseChapterItems(tocText, opfText) {
      var parser = new DOMParser();
      var tocXml = parser.parseFromString(tocText, 'application/xml');
      var opfXml = parser.parseFromString(opfText, 'application/xml');
      var tocErrorNode = tocXml.querySelector('parsererror');
      var opfErrorNode = opfXml.querySelector('parsererror');

      if (tocErrorNode) {
        throw new Error('toc.ncx 解析失败');
      }

      if (opfErrorNode) {
        throw new Error('content.opf 解析失败');
      }

      var navMap = tocXml.getElementsByTagName('navMap')[0];
      var navPoints = getElementChildrenByLocalName(navMap, 'navPoint');
      var navItems = navPoints.map(function (navPoint) {
        return parseNavPoint(navPoint);
      });
      var manifestMap = buildManifestMap(opfXml);
      var spineItems = buildSpineItems(opfXml, manifestMap);

      return mergeTocWithSpine(navItems, spineItems);
    }

    function buildChapterBasePath(novelName, chapter) {
      return '/novels/' + novelName + '/' + chapter + '/OEBPS';
    }

    function getNovelControls(card) {
      return {
        controls: card.querySelector('[data-role="inline-controls"]'),
        status: card.querySelector('[data-role="inline-reader-status"]'),
        frame: card.querySelector('[data-role="chapter-frame"]')
      };
    }

    function setReaderStatus(card, message, isError) {
      var parts = getNovelControls(card);

      if (!parts.status) {
        return;
      }

      parts.status.textContent = message || '';
      parts.status.setAttribute('data-error', isError ? 'true' : 'false');
    }

    function getCurrentFrameDocument() {
      var parts;

      if (!state.currentNovelCard) {
        return null;
      }

      parts = getNovelControls(state.currentNovelCard);
      return getFrameDocument(parts.frame);
    }

    function resizeFrame(frame) {
      var frameDocument;
      var frameBody;
      var frameHtml;
      var height;
      var width;

      if (!frame) {
        return;
      }

      try {
        frameDocument = frame.contentDocument || (frame.contentWindow && frame.contentWindow.document);
      } catch (error) {
        return;
      }

      if (!frameDocument) {
        return;
      }

      frameBody = frameDocument.body;
      frameHtml = frameDocument.documentElement;

      if (!frameBody || !frameHtml) {
        return;
      }

      frameBody.style.overflowX = 'auto';
      frameBody.style.overflowY = 'hidden';
      frameHtml.style.overflowX = 'auto';
      frameHtml.style.overflowY = 'hidden';

      width = Math.max(
        frameBody.scrollWidth,
        frameBody.offsetWidth,
        frameHtml.clientWidth,
        frameHtml.scrollWidth,
        frameHtml.offsetWidth
      );

      height = Math.max(
        frameBody.scrollHeight,
        frameBody.offsetHeight,
        frameHtml.clientHeight,
        frameHtml.scrollHeight,
        frameHtml.offsetHeight
      );

      frame.style.height = height + 'px';
      frame.style.width = '100%';
      frame.style.maxWidth = '100%';
    }

    function updatePagerButtons(card) {
      var prevButton = card.querySelector('[data-action="prev-page"]');
      var nextButton = card.querySelector('[data-action="next-page"]');

      if (prevButton) {
        prevButton.disabled = state.currentIndex <= 0;
      }

      if (nextButton) {
        nextButton.disabled = state.currentIndex < 0 || state.currentIndex >= state.currentItems.length - 1;
      }
    }

    function buildControlsHtml(novel) {
      var html = '';
      var i;
      var chapter;

      html += '<div>';
      html += '<button type="button" data-action="back-to-list" style="background: transparent; border: 1px solid currentColor;">返回列表</button>';
      html += '<div data-role="chapter-dropdown" style="display: inline-block;">';
      html += '<button type="button" data-role="chapter-dropdown-toggle">选择章节</button>';
      html += '<div data-role="chapter-dropdown-menu" hidden>';

      for (i = 0; i < novel.chapters.length; i += 1) {
        chapter = novel.chapters[i];
        html += '<button type="button" data-role="chapter-option" data-value="' + escapeHtml(chapter) + '">第' + escapeHtml(chapter) + '章</button>';
      }

      html += '</div>';
      html += '</div>';
      html += '<button type="button" data-action="prev-page" style="margin-left: 12px; background: transparent; border: 1px solid currentColor;">上一页</button>';
      html += '<button type="button" data-action="next-page" style="margin-left: 8px; background: transparent; border: 1px solid currentColor;">下一页</button>';
      html += '</div>';

      return html;
    }

    function getReaderThemeCss() {
      var darkMode = !!(window.blog && window.blog.darkMode);

      if (darkMode) {
        return [
          ':root { color-scheme: dark; }',
          'html, body {',
          '  max-width: 100%;',
          '  overflow-x: auto;',
          '  overflow-y: hidden;',
          '  background: #0d1117 !important;',
          '  color: #c9d1d9 !important;',
          '}',
          'html[data-theme-transition], html[data-theme-transition] body { transition: all 500ms ease; }',
          'body, p, div, span, li, dd, dt, blockquote, article, section, main, aside, header, footer { color: #c9d1d9 !important; }',
          'a { color: #c9d1d9 !important; text-decoration: none !important; border-bottom: 1px solid #c9d1d9 !important; }',
          'a::after { display: none !important; }',
          'img { max-width: 100%; height: auto; }',
          '.duokan-image-single, .kuan, .zhai { text-align: center !important; overflow-x: auto; }',
          '.duokan-image-single img, .kuan img, .zhai img { display: inline-block; }'
        ].join('\n');
      }

      return [
        ':root { color-scheme: light; }',
        'html, body {',
        '  max-width: 100%;',
        '  overflow-x: auto;',
        '  overflow-y: hidden;',
        '  background: #FFFFFF !important;',
        '  color: #333333 !important;',
        '}',
        'html[data-theme-transition], html[data-theme-transition] body { transition: all 500ms ease; }',
        'body, p, div, span, li, dd, dt, blockquote, article, section, main, aside, header, footer { color: #333333 !important; }',
        'a { color: #333333 !important; text-decoration: none !important; border-bottom: 1px solid #333333 !important; }',
        'a::after { display: none !important; }',
        'img { max-width: 100%; height: auto; }',
        '.duokan-image-single, .kuan, .zhai { text-align: center !important; overflow-x: auto; }',
        '.duokan-image-single img, .kuan img, .zhai img { display: inline-block; }'
      ].join('\n');
    }

    function applyReaderTheme(frameDocument) {
      var styleElement;
      var head;
      var body;
      var html;

      if (!frameDocument) {
        return;
      }

      head = frameDocument.head;
      body = frameDocument.body;
      html = frameDocument.documentElement;

      if (!head || !body || !html) {
        return;
      }

      styleElement = frameDocument.getElementById('novels-reader-theme');

      if (!styleElement) {
        styleElement = frameDocument.createElement('style');
        styleElement.id = 'novels-reader-theme';
        head.appendChild(styleElement);
      }

      styleElement.textContent = getReaderThemeCss();
      html.style.colorScheme = window.blog && window.blog.darkMode ? 'dark' : 'light';
    }

    function setReaderThemeTransition(frameDocument, enabled) {
      var html;

      if (!frameDocument) {
        return;
      }

      html = frameDocument.documentElement;

      if (!html) {
        return;
      }

      if (enabled) {
        html.setAttribute('data-theme-transition', '');
        return;
      }

      html.removeAttribute('data-theme-transition');
    }

    function syncCurrentReaderThemeWithSiteTransition() {
      var parts;
      var frameDocument;

      if (!state.currentNovelCard) {
        return;
      }

      parts = getNovelControls(state.currentNovelCard);

      if (!parts.frame) {
        return;
      }

      frameDocument = getFrameDocument(parts.frame);

      if (!frameDocument) {
        return;
      }

      setReaderThemeTransition(frameDocument, true);
      refreshCurrentReaderTheme();

      window.setTimeout(function () {
        setReaderThemeTransition(frameDocument, false);
      }, 600);
    }

    function refreshCurrentReaderTheme() {
      var parts;
      var frameDocument;

      if (!state.currentNovelCard) {
        return;
      }

      parts = getNovelControls(state.currentNovelCard);

      if (!parts.frame) {
        return;
      }

      frameDocument = getFrameDocument(parts.frame);

      if (!frameDocument) {
        return;
      }

      applyReaderTheme(frameDocument);
      resizeFrame(parts.frame);
    }

    function resetOtherNovels(activeCard) {
      forEachNode(novelCards, function (card) {
        var parts = getNovelControls(card);

        if (card === activeCard) {
          card.hidden = false;
          return;
        }

        card.hidden = true;

        if (parts.controls) {
          parts.controls.hidden = true;
          parts.controls.innerHTML = '';
        }

        if (parts.status) {
          parts.status.textContent = '';
          parts.status.setAttribute('data-error', 'false');
        }

        resetFrame(parts.frame);
      });
    }

    function resetToNovelList() {
      state.chapterRequestId += 1;

      forEachNode(novelCards, function (card) {
        var parts = getNovelControls(card);

        card.hidden = false;

        if (parts.controls) {
          parts.controls.hidden = true;
          parts.controls.innerHTML = '';
          parts.controls.removeAttribute('data-sticky');
        }

        if (parts.status) {
          parts.status.textContent = '';
          parts.status.setAttribute('data-error', 'false');
        }

        resetFrame(parts.frame);
      });

      state.currentNovelCard = null;
      state.currentNovel = null;
      state.currentChapter = null;
      state.currentItems = [];
      state.currentIndex = -1;
    }

    function renderNovelControls(card, novel) {
      var parts = getNovelControls(card);
      var toggle;
      var firstOption;
      var dropdown;
      var options;
      var maxWidth = 0;
      var measure;

      if (!parts.controls) {
        return;
      }

      parts.controls.innerHTML = buildControlsHtml(novel);
      parts.controls.hidden = false;

      toggle = parts.controls.querySelector('[data-role="chapter-dropdown-toggle"]');
      firstOption = parts.controls.querySelector('[data-role="chapter-option"]');
      dropdown = parts.controls.querySelector('[data-role="chapter-dropdown"]');
      options = parts.controls.querySelectorAll('[data-role="chapter-option"]');

      if (toggle && firstOption && novel.chapters.length) {
        toggle.textContent = firstOption.textContent;
        firstOption.setAttribute('data-selected', 'true');
      }

      if (dropdown && toggle && options.length) {
        measure = document.createElement('button');
        measure.type = 'button';
        measure.setAttribute('data-role', 'chapter-option');
        measure.style.position = 'absolute';
        measure.style.visibility = 'hidden';
        measure.style.pointerEvents = 'none';
        measure.style.whiteSpace = 'nowrap';
        measure.style.width = 'auto';
        measure.style.maxWidth = 'none';

        parts.controls.appendChild(measure);

        forEachNode(options, function (option) {
          measure.textContent = option.textContent;
          maxWidth = Math.max(maxWidth, measure.offsetWidth);
        });

        measure.textContent = toggle.textContent;
        maxWidth = Math.max(maxWidth, measure.offsetWidth, toggle.offsetWidth);

        parts.controls.removeChild(measure);

        if (maxWidth > 0) {
          dropdown.style.width = maxWidth + 'px';
        }
      }

      resetFrame(parts.frame);
    }

    function updateStickyControls() {
      var parts;
      var rect;

      if (!state.currentNovelCard) {
        return;
      }

      parts = getNovelControls(state.currentNovelCard);

      if (!parts.controls || parts.controls.hidden) {
        return;
      }

      rect = parts.controls.getBoundingClientRect();

      if (rect.top <= 0) {
        parts.controls.setAttribute('data-sticky', 'true');
      } else {
        parts.controls.removeAttribute('data-sticky');
      }
    }

    function openReaderAt(index) {
      var parts;
      var item;
      var xhtmlUrl;

      if (!state.currentNovelCard || !state.currentChapter) {
        return;
      }

      if (!state.currentItems.length || index < 0 || index >= state.currentItems.length) {
        return;
      }

      parts = getNovelControls(state.currentNovelCard);

      if (!parts.frame) {
        return;
      }

      state.currentIndex = index;
      item = state.currentItems[index];
      xhtmlUrl = state.currentChapter.basePath + '/' + item.src;

      setReaderStatus(state.currentNovelCard, '', false);

      if (parts.frame.parentNode) {
        parts.frame.parentNode.hidden = false;
      }

      parts.frame.hidden = false;
      parts.frame.style.height = '0px';
      parts.frame.src = encodeURI(xhtmlUrl);
      updatePagerButtons(state.currentNovelCard);
      updateStickyControls();
    }

    function openChapter(chapter) {
      var basePath;
      var tocUrl;
      var opfUrl;
      var requestId;
      var activeCard;

      if (!state.currentNovel || !state.currentNovelCard) {
        return;
      }

      activeCard = state.currentNovelCard;
      requestId = state.chapterRequestId + 1;
      state.chapterRequestId = requestId;

      basePath = buildChapterBasePath(state.currentNovel.name, chapter);
      tocUrl = basePath + '/toc.ncx';
      opfUrl = basePath + '/content.opf';

      state.currentChapter = {
        chapter: chapter,
        basePath: basePath
      };
      state.currentItems = [];
      state.currentIndex = -1;

      updateChapterSelection(activeCard, chapter);
      closeDropdown(activeCard);
      resetFrame(getNovelControls(activeCard).frame);
      setReaderStatus(activeCard, '章节内容加载中...', false);
      updatePagerButtons(activeCard);

      Promise.all([
        fetch(encodeURI(tocUrl)).then(function (response) {
          if (!response.ok) {
            throw new Error('toc.ncx HTTP ' + response.status);
          }

          return response.text();
        }),
        fetch(encodeURI(opfUrl)).then(function (response) {
          if (!response.ok) {
            throw new Error('content.opf HTTP ' + response.status);
          }

          return response.text();
        })
      ])
        .then(function (results) {
          var items;

          if (state.chapterRequestId !== requestId || state.currentNovelCard !== activeCard) {
            return;
          }

          items = parseChapterItems(results[0], results[1]);

          if (!items.length) {
            throw new Error('未找到可阅读内容');
          }

          state.currentItems = items;
          openReaderAt(0);
        })
        .catch(function (error) {
          if (state.chapterRequestId !== requestId || state.currentNovelCard !== activeCard) {
            return;
          }

          setReaderStatus(activeCard, '章节加载失败：' + error.message, true);
          updatePagerButtons(activeCard);
        });
    }

    function openNovel(card, novel) {
      state.currentNovelCard = card;
      state.currentNovel = novel;
      state.currentChapter = null;
      state.currentItems = [];
      state.currentIndex = -1;

      resetOtherNovels(card);
      renderNovelControls(card, novel);

      if (novel.chapters && novel.chapters.length) {
        openChapter(novel.chapters[0]);
      } else {
        setReaderStatus(card, '暂无章节', false);
        updatePagerButtons(card);
      }
    }

    app.addEventListener('click', function (event) {
      var novelLink = event.target.closest('[data-action="open-novel"]');
      var actionButton = event.target.closest('[data-action]');
      var dropdownToggle = event.target.closest('[data-role="chapter-dropdown-toggle"]');
      var chapterOption = event.target.closest('[data-role="chapter-option"]');
      var currentDropdown;
      var currentMenu;
      var novelCard;
      var chapters = [];
      var action;

      if (novelLink) {
        event.preventDefault();

        novelCard = novelLink.closest('[data-role="novel-card"]');

        if (!novelCard) {
          return;
        }

        if (state.currentNovelCard === novelCard) {
          resetToNovelList();
          return;
        }

        try {
          chapters = JSON.parse(novelCard.getAttribute('data-chapters') || '[]');
        } catch (error) {
          chapters = [];
        }

        openNovel(novelCard, {
          name: novelCard.getAttribute('data-novel-name') || '',
          author: novelCard.getAttribute('data-novel-author') || '',
          chapters: chapters
        });
        return;
      }

      if (dropdownToggle) {
        event.preventDefault();
        currentDropdown = dropdownToggle.closest('[data-role="chapter-dropdown"]');
        currentMenu = currentDropdown && currentDropdown.querySelector('[data-role="chapter-dropdown-menu"]');

        if (currentMenu) {
          currentMenu.hidden = !currentMenu.hidden;
        }

        return;
      }

      if (chapterOption && state.currentNovelCard) {
        event.preventDefault();
        openChapter(chapterOption.getAttribute('data-value') || '');
        return;
      }

      if (!actionButton) {
        return;
      }

      action = actionButton.getAttribute('data-action');

      if (action === 'prev-page') {
        event.preventDefault();

        if (state.currentIndex > 0) {
          openReaderAt(state.currentIndex - 1);
        }

        return;
      }

      if (action === 'next-page') {
        event.preventDefault();

        if (state.currentIndex < state.currentItems.length - 1) {
          openReaderAt(state.currentIndex + 1);
        }

        return;
      }

      if (action === 'back-to-list') {
        event.preventDefault();
        resetToNovelList();
      }
    });

    forEachNode(novelCards, function (card) {
      var parts = getNovelControls(card);

      resetFrame(parts.frame);

      if (parts.frame) {
        parts.frame.addEventListener('load', function () {
          var frameDocument = getFrameDocument(parts.frame);

          if (card !== state.currentNovelCard) {
            return;
          }

          if (frameDocument) {
            applyReaderTheme(frameDocument);
          }

          resizeFrame(parts.frame);
          setReaderStatus(card, '', false);
        });
      }
    });

    window.addEventListener('resize', function () {
      if (!state.currentNovelCard) {
        return;
      }

      resizeFrame(getNovelControls(state.currentNovelCard).frame);
      updateStickyControls();
    });

    window.addEventListener('scroll', function () {
      updateStickyControls();
    });

    document.addEventListener('click', function (event) {
      if (event.target && event.target.closest('.footer-btn.theme-toggler')) {
        syncCurrentReaderThemeWithSiteTransition();
      } else if (!event.target.closest('[data-role="chapter-dropdown"]') && state.currentNovelCard) {
        closeDropdown(state.currentNovelCard);
      }
    });

    if (window.matchMedia) {
      if (window.matchMedia('(prefers-color-scheme: dark)').addEventListener) {
        window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', function () {
          syncCurrentReaderThemeWithSiteTransition();
        });
      } else {
        window.matchMedia('(prefers-color-scheme: dark)').addListener(function () {
          syncCurrentReaderThemeWithSiteTransition();
        });
      }
    }
  })();
</script>
