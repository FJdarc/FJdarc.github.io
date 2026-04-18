---
layout: mypost
title: 小说
---

<style>
  [data-role="inline-controls"][data-sticky="true"] {
    position: sticky;
    top: 0;
    z-index: 10;
    background: #fff;
    padding: 8px 0;
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
            <a href="#" data-action="open-novel">{{ novel.name }}</a>
            <span style="float: right;">{{ novel.author }}</span>
          </p>
          <div data-role="inline-controls" hidden></div>
          <div data-role="inline-reader-status"></div>
          <div data-role="inline-reader">
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
      currentIndex: -1
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

    function resizeFrame(frame) {
      var frameDocument;
      var frameBody;
      var frameHtml;
      var height;

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

      frameBody.style.overflow = 'hidden';
      frameHtml.style.overflow = 'hidden';

      height = Math.max(
        frameBody.scrollHeight,
        frameBody.offsetHeight,
        frameHtml.clientHeight,
        frameHtml.scrollHeight,
        frameHtml.offsetHeight
      );

      frame.style.height = height + 'px';
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
      html += '<button type="button" data-action="back-to-list">返回列表</button>';
      html += '<select data-role="chapter-select" style="margin-left: 12px;">';

      for (i = 0; i < novel.chapters.length; i += 1) {
        chapter = novel.chapters[i];
        html += '<option value="' + escapeHtml(chapter) + '">第' + escapeHtml(chapter) + '章</option>';
      }

      html += '</select>';
      html += '<button type="button" data-action="prev-page" style="margin-left: 12px;">上一页</button>';
      html += '<button type="button" data-action="next-page" style="margin-left: 8px;">下一页</button>';
      html += '</div>';

      return html;
    }

    function resetOtherNovels(activeCard) {
      Array.prototype.forEach.call(novelCards, function (card) {
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

        if (parts.frame) {
          parts.frame.hidden = true;
          parts.frame.removeAttribute('src');
          parts.frame.style.height = '0px';
          parts.frame.style.width = '100%';
          parts.frame.style.display = 'block';
          parts.frame.style.border = '0';
          parts.frame.style.overflow = 'hidden';
        }
      });
    }

    function resetToNovelList() {
      Array.prototype.forEach.call(novelCards, function (card) {
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

        if (parts.frame) {
          parts.frame.hidden = true;
          parts.frame.removeAttribute('src');
          parts.frame.style.height = '0px';
        }
      });

      state.currentNovelCard = null;
      state.currentNovel = null;
      state.currentChapter = null;
      state.currentItems = [];
      state.currentIndex = -1;
    }

    function renderNovelControls(card, novel) {
      var parts = getNovelControls(card);
      var select;

      if (!parts.controls) {
        return;
      }

      parts.controls.innerHTML = buildControlsHtml(novel);
      parts.controls.hidden = false;

      select = parts.controls.querySelector('[data-role="chapter-select"]');

      if (select && novel.chapters.length) {
        select.value = String(novel.chapters[0]);
      }

      if (parts.frame) {
        parts.frame.style.width = '100%';
        parts.frame.style.display = 'block';
        parts.frame.style.border = '0';
        parts.frame.style.overflow = 'hidden';
        parts.frame.style.height = '0px';
      }
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
      var select;

      if (!state.currentNovel || !state.currentNovelCard) {
        return;
      }

      basePath = buildChapterBasePath(state.currentNovel.name, chapter);
      tocUrl = basePath + '/toc.ncx';
      opfUrl = basePath + '/content.opf';

      state.currentChapter = {
        chapter: chapter,
        basePath: basePath
      };
      state.currentItems = [];
      state.currentIndex = -1;

      select = state.currentNovelCard.querySelector('[data-role="chapter-select"]');

      if (select) {
        select.value = String(chapter);
      }

      setReaderStatus(state.currentNovelCard, '章节内容加载中...', false);
      updatePagerButtons(state.currentNovelCard);

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
          var items = parseChapterItems(results[0], results[1]);

          if (!items.length) {
            throw new Error('未找到可阅读内容');
          }

          state.currentItems = items;
          openReaderAt(0);
        })
        .catch(function (error) {
          setReaderStatus(state.currentNovelCard, '章节加载失败：' + error.message, true);
          updatePagerButtons(state.currentNovelCard);
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
      var novelCard;
      var chapters = [];

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

      if (!actionButton) {
        return;
      }

      if (actionButton.getAttribute('data-action') === 'prev-page') {
        event.preventDefault();

        if (state.currentIndex > 0) {
          openReaderAt(state.currentIndex - 1);
        }

        return;
      }

      if (actionButton.getAttribute('data-action') === 'next-page') {
        event.preventDefault();

        if (state.currentIndex < state.currentItems.length - 1) {
          openReaderAt(state.currentIndex + 1);
        }

        return;
      }

      if (actionButton.getAttribute('data-action') === 'back-to-list') {
        event.preventDefault();
        resetToNovelList();
      }
    });

    app.addEventListener('change', function (event) {
      var chapterSelect = event.target.closest('[data-role="chapter-select"]');

      if (!chapterSelect || !state.currentNovelCard) {
        return;
      }

      openChapter(chapterSelect.value || '');
    });

    Array.prototype.forEach.call(novelCards, function (card) {
      var parts = getNovelControls(card);

      if (parts.frame) {
        parts.frame.style.width = '100%';
        parts.frame.style.display = 'block';
        parts.frame.style.border = '0';
        parts.frame.style.overflow = 'hidden';

        parts.frame.addEventListener('load', function () {
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
  })();
</script>