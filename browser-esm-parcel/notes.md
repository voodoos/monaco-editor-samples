Issue : the browser-esm-parcel sample was only working for features not requiring workers.

When trying to enable typescript (and thus the typescript worker) the following message appeared in the console :

```
Could not create web worker(s). Falling back to loading web worker
code in main thread, which might cause UI freezes. 
Please see https://github.com/Microsoft/monaco-editor#faq 

Error: Cannot find module 'vs/language/typescript/tsWorker'
Stack trace:
	[...]
```

Solution ? :
This is in part due to a weird behavior of Parcel when confronted to web workers. For details see this (long-opened) issue : https://github.com/parcel-bundler/parcel/issues/670

One solution is to make a local copy of the web-worker entry points from monaco and correct the links inside.

The `getWorker` function becomes :

```js
self.MonacoEnvironment  = {
	getWorker:  function (moduleId, label) {
		if (label  ===  'json') {
			return  new  Worker('./workers/json.worker.js')
		}
		if (label  ===  'css') {
			return  new  Worker('./workers/css.worker.js')
		}
		if (label  ===  'html') {
			return  new  Worker('./workers/html.worker.js')
		}
		if (label  ===  'typescript'  
			||  label  ===  'javascript') {
			return  new  Worker('./workers/ts.worker.js')
		}
		return  new  Worker('./workers/editor.worker.js')
	}
}
```

It works for the `editor.worker` but the problem persist for the other workers.