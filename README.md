[Info]  
> This google chrome extention was made to help me bypass my schools edoctrina assignments.
> Edoctrina uses a extention to "lock" and "log" actions in the testing window.

I put quotes around lock and log because the extention itself doesn't do anything besides send notifications lol  

[Explaining the code]
The code below is functions to determine the right url.
```js
  function isAssessmentUrl(url) {
    const pattern = /^http[s]?[:][/]{2}.+?[.]edoctrina[.]org[/]v2[/]take-online-test[.]html.*$/g;
    return pattern.test(url);
  }

  function isAllowedUrl(url) {
    const pattern = /^(file:[/]{2}|chrome:[/]{2}|http[s]?[:][/]{2}.+?[.]edoctrina[.]org.*)/g;
    return pattern.test(url);
  }
```

Close all tabs function which closes all the tabs that isnt the valid url
```js

  function closeAllTabs() {
    chrome.windows.getAll({
      populate: true
    }, (windows) => {
      if (windows.length) {
        let closedLength = 0;
        for (let windowIndex in windows) {
          let currentWindow = windows[windowIndex];
          if ((typeof (currentWindow['tabs']) != 'undefined') && currentWindow['tabs'].length) {
            for (let tabIndex in currentWindow['tabs']) {
              let tab = currentWindow['tabs'][tabIndex];
              if (!isAllowedUrl(tab.url)) {
                closedLength++;
                chrome.tabs.remove(tab.id);
              }
            }
          }
        }
        if (closedLength > 0) {
          const opt = {
            type: 'basic',
            title: 'This Assessment is Locked',
            message: 'You are not allowed to open any other websites while taking an online assessment. Your attempt was registered.',
            iconUrl: 'icon/icon-128.png'
          };
          chrome.notifications.create('locker', opt);
        }
      }
    });
  }
```

