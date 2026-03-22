# xcrun mcpbridge tools/list

Tracks Xcode's built-in MCP server `tools/list` responses across versions.

## How to capture a response

Use [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector):

- **Transport Type**: `STDIO`
- **Command:** `/usr/bin/xcrun`
- **Arguments:** `mcpbridge`

Then invoke `tools/list` and save the response JSON.

## Directory structure

Each Xcode version gets a directory named `xcode-{version}-{release-type}-{build}`:

```
xcode-26.4-rc-17E192/
└── tools-list.json    # Raw tools/list response
```
