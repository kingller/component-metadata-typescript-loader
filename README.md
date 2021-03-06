# component-metadata-ts-loader
A Webpack loader for extracting React Component metadata (props, comments jsDoclets, etc) defined in TypeScript. 
Helpful for get documentation information from React components, it uses 
[react-docgen-typescript](https://github.com/styleguidist/react-docgen-typescript) 
to parse and return JSON metadata when requiring a file.

## Installation

```sh
$ npm install --save component-metadata-ts-loader
```

## Usage

Generally you will want to use the inline request syntax for using this loader,
instead of adding it to your config file.

```js
var metadata = require('component-metadata-ts-loader!./some/my-component');

metadata.componentDocs[0] // { props, description, displayName }
```

### Instruction

The loader will parse out any jsDoc style from either component or propType comment blocks. You can
access them from `metadata.componentDocs[0].props`

- `@default`: for manually specifying a default value for a prop.

### Exporting Components

**It is important** to export your component using a named export for docgen information to be generated properly.

---

`TicTacToeCell.tsx`:

```javascript
import React, { Component } from "react";
import * as styles from "./TicTacToeCell.css";

interface Props {
  /**
   * Value to display, either empty (" ") or "X" / "O".
   *
   * @default " "
   **/
  value?: " " | "X" | "O";

  /** Cell position on game board. */
  position: { x: number, y: number };

  /** Called when an empty cell is clicked. */
  onClick?: (x: number, y: number) => void;
}

/**
 * TicTacToe game cell. Displays a clickable button when the value is " ",
 * otherwise displays "X" or "O".
 */
// Notice the named export here, this is required for docgen information
// to be generated correctly.
export class TicTacToeCell extends Component<Props> {
  handleClick = () => {
    const {
      position: { x, y },
      onClick,
    } = this.props;
    if (!onClick) return;

    onClick(x, y);
  };

  render() {
    const { value = " " } = this.props;
    const disabled = value !== " ";
    const classes = `${styles.button} ${disabled ? styles.disabled : ""}`;

    return (
      <button
        className={classes}
        disabled={disabled}
        onClick={this.handleClick}
      >
        {value}
      </button>
    );
  }
}

// Component can still be exported as default.
export default TicTacToeCell;
```

Will generate the following outpout: 
```json
{
  "componentDocs": [{
    "description": "TicTacToe game cell. Displays a clickable button when the value is \" \",\notherwise displays \"X\" or \"O\".",
    "displayName": "TicTacToeCell",
    "methods": [],
    "props": {
      "value": {
        "defaultValue": {
          "value": "\" \""
        },
        "description": "Value to display, either empty (\" \") or \"X\" / \"O\".",
        "name": "value",
        "required": false,
        "type": {
          "name": "\" \" | \"X\" | \"O\""
        }
      },
      "position": {
        "defaultValue": null,
        "description": "Cell position on game board.",
        "name": "position",
        "required": true,
        "type": {
          "name": "{ x: number; y: number; }"
        }
      },
      "onClick": {
        "defaultValue": null,
        "description": "Called when an empty cell is clicked.",
        "name": "onClick",
        "required": false,
        "type": {
          "name": "(x: number, y: number) => void"
        }
      }
    }
  }]
}
```

## Limitations

This plugin makes use of the project:
https://github.com/styleguidist/react-docgen-typescript.
It is subject to the same limitations.

### Export Names

Component docgen information can not be
generated for components that are only exported as default. You can work around
the issue by exporting the component using a named export.

```javascript
import * as React from "react";

interface ColorButtonProps {
  /** Buttons background color */
  color: "blue" | "green";
}

/** A button with a configurable background color. */
export const ColorButton: React.SFC<ColorButtonProps> = props => (
  <button
    style={{
      padding: 40,
      color: "#eee",
      backgroundColor: props.color,
      fontSize: "2rem",
    }}
  >
    {props.children}
  </button>
);

export default ColorButton;
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.
