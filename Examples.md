### Example 1 ###

Following is a contrived and condensed pseudo-example based only on functions added to the history stack. It doesn't mess with the visitor's window hash at all, and it's done to show that you don't have to mess with the URI at all if you don't want to for some reason.

```
<body>
  <script src="dshistory.js"></script>

  <div id="first-tab" onclick="showContent({contentToShow: 'first-block-content', contentToHide: 'second-block-content'})">First tab</div>
  <div id="second-tab" onclick="showContent({contentToShow: 'second-block-content', contentToHide: 'first-block-content'})">Second tab</div>

  <div id="first-block-content">First block content</div>
  <div id="second-block-content" style="display: none;">Second block content</div>

  <script>
  function showContent(contentObject, historyObject) {
    if (!historyObject || !historyObject.calledFromHistory) {
        dsHistory.addFunction(showContent, window, contentObject);
    }

    document.getElementById(contentObject.contentToShow).style.display = 'block';
    document.getElementById(contentObject.contentToHide).style.display = 'none';
  }
  
  dsHistory.addFunction(showContent, window, {
    contentToShow: 'first-block-content',
    contentToHide: 'second-block-content'
  });
  </script>
</body>
```

### Example 2 ###

Now, let's say you want to do the same thing but support bookmarking. You can do something like this:

```
<body>
  <script src="dshistory.js"></script>

  <div id="first-tab" onclick="showContent({contentToShow: 'first-block-content', contentToHide: 'second-block-content'})">First tab</div>
  <div id="second-tab" onclick="showContent({contentToShow: 'second-block-content', contentToHide: 'first-block-content'})">Second tab</div>

  <div id="first-block-content">First block content</div>
  <div id="second-block-content" style="display: none;">Second block content</div>

  <script>
  function showContent(contentObject, historyObject) {
    if (!historyObject || !historyObject.calledFromHistory) {
        dsHistory.setQueryVar('Showing', contentObject.contentToShow);
        dsHistory.setQueryVar('Hiding', contentObject.contentToHide);
        // this should be treated the same as addFunction as the setQueryVar functions above really don't have any impact yet
        dsHistory.bindQueryVars(showContent, window, contentObject);
        // now, dsHistory.QueryElements['Showing'] = contentObject.contentToShow;
        // now, dsHistory.QueryElements['Hiding'] = contentObject.contentToHide;
    }

    document.getElementById(contentObject.contentToShow).style.display = 'block';
    document.getElementById(contentObject.contentToHide).style.display = 'none';
  }
  
  if (typeof dsHistory.QueryElements['Showing'] == 'string' && typeof dsHistory.QueryElements['Hiding'] == 'string') {
    showContent({contentToShow: dsHistory.QueryElements['Showing'], contentToHide: dsHistory.QueryElements['Hiding']});
  } else {
    dsHistory.addFunction(showContent, window, {
      contentToShow: 'first-block-content',
      contentToHide: 'second-block-content'
    });
  }
  </script>
</body>
```

### Working Demo ###

If you want to see a working demo, go ahead and [take a look](http://dshistory.googlecode.com/svn/trunk/examples/demo.html).