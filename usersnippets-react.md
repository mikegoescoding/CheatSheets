# React User Snippets vscode

---

- bottom left gear icon > user snippets
- find & open: JavaScript React 'javascriptreact.json' file

---

## Example snippets (paste to json file):

#### "prefix" is where you define whatever you want to call your shortcut key

```
{
	"Create a functional react component": {
		"prefix": "rfc",
		"body": [
			"import React from 'react';",
			"",
			"export default function $1() {",
			"	return (",
			"		<div className='$2'>",
			"			$3",
			"		</div>",
			"	);",
			"}"
		],
		"description": "creates the skeleton for a functional component in react"
	},
	"Create a react component with constructor": {
		"prefix": "irc",
		"body": [
			"import React, { Component } from 'react';",
			"",
			"export default class $1namethisclass extends Component {",
			"	constructor(props) {",
			"		super(props);",
			"",
			"		this.state = {",
			"			$2",
			"		};",
			"	}",
			"",
			"	render() {",
			"		return (",
			"			<div className='$2'>",
			"				$3",
			"			</div>",
			"		);",
			"	}"
			"}"
		],
		"description": "creates the skeleton for default component in react"
	},
	"shortcut for local host": {
		"prefix": "127",
		"body": [
			"127.0.0.1:5000"
		],
		"description": "shortcut for local host"
	}
}
```
