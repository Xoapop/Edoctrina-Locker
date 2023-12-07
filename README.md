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
Function to check all url and determin if its a valid url or not
```js
  function checkOpenTabs(callback) {
    chrome.windows.getAll({
      populate: true
    }, (windows) => {
      let result = {
        isAssessmentOpen: false,
        closeTabs: false,
        isNotAvailableOpen: false
      };
      let queue = 0;
      if (windows.length > 0) {
        for (let windowIndex in windows) {
          let currentWindow = windows[windowIndex];
          if ((typeof (currentWindow['tabs']) != 'undefined') && currentWindow['tabs'].length) {
            for (let tabIndex in currentWindow['tabs']) {
              let tab = currentWindow['tabs'][tabIndex];
              if (isAssessmentUrl(tab.url)) {
                queue++;
                chrome.tabs.sendMessage(tab.id, {
                  text: 'is_test_lock'
                }, {}, (data) => {
                  queue--;
                  if (data && (typeof (data['is_test_lock']) != 'undefined') && data['is_test_lock'] == "1") {
                    result.isAssessmentOpen = true;
                  }
                });
                chrome.tabs.sendMessage(tab.id, {
                  text: 'is_student'
                }, {}, (data) => {
                  if (data && (typeof (data['is_student']) != 'undefined') && data['is_student']) {
                    result.closeTabs = true;
                  }
                });
              } else {
                if (!isAllowedUrl(tab.url)) {
                  result.isNotAvailableOpen = true;
                }
              }
            }
          }
        }
      }
      setTimeout(() => {
        if (queue > 0) {
          setTimeout(() => {}, 500);
        } else {
          callback(result);
        }
      }, 500);
    });
  }
```
Rest of the functions are what makes the extention it self run
```js
  function actionCheck() {
    checkOpenTabs((result) => {
      if (result.isAssessmentOpen && result.closeTabs) {
        closeAllTabs();
      }
    });
  }

  chrome.tabs.onActivated.addListener(() => actionCheck());
  chrome.tabs.onUpdated.addListener(() => actionCheck());
  chrome.windows.onFocusChanged.addListener(() => actionCheck());
})();
```

[More explanation]  
If you can read the code correctly the file it self doesn't log shit besides the sending a simple notification and close all tabs that aren't valid urls.  
The fix for this is removing the part in the close all tabs function that sends the notification about it being registered and also removing the part where function that removes the tab if it isnt a valid url.  

The files i've provided in the repo fixes this issue. All you have to do is learn how to import it into your browser.
