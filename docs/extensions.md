# Scratch 3.0擴充功能（Scratch 3.0 Extensions）

本文描述與Scratch 3.0擴充功能開發相關的科技議題，包含Scratch 3.0擴充功能的規格。

This document describes technical topics related to Scratch 3.0 extension development, including the Scratch 3.0
extension specification.

有關於其他與Scratch 3.0擴充功能相關的部分，請參考[this Extensions page on the wiki](https://github.com/LLK/docs/wiki/Extensions)。

For documentation regarding other aspects of Scratch 3.0 extensions see [this Extensions page on the
wiki](https://github.com/LLK/docs/wiki/Extensions).

## 擴充功能的類型（Types of Extensions）

Scratch 3.0有四種類型的擴充功能，包含Scratch核心函式庫（例如：「外觀」與「運算」類型）到可以從遠端網址下載的非官方擴充功能。

There are four types of extensions that can define everything from the Scratch's core library (such as the "Looks" and
"Operators" categories) to unofficial extensions that can be loaded from a remote URL.

**Scratch 3.0至今並不支持非官方的擴充功能。**

**Scratch 3.0 does not yet support unofficial extensions.**

更多相關訊息，詳見[this Extensions page on the wiki](https://github.com/LLK/docs/wiki/Extensions)。

For more details, see [this Extensions page on the wiki](https://github.com/LLK/docs/wiki/Extensions).



|                                                    | Core（核心）  | Team（團隊 ） | Official（官方） | Unofficial（非官方） |
| ----------------------------------------------     | ------------ | ------------| ---------------  | ------------------  |
| 由Scratch團隊開發（Developed by Scratch Team）      |      √        |      √      |        O         |          X          |
| 由Scratch團隊維護（Maintained by Scratch Team）     |      √        |      √      |        O         |          X          |
| 出現於擴充功能區（Shown in Library）                |      X        |       √      |        √         |           X          |
| 實驗擴充功能（Sandboxed）                           |      X        |       X      |        √         |           √          |
| 能夠儲存至社群專案（Can save projects to community） |     √        |     √        |       √          |           X          |

## JavaScript開發環境（JavaScript Environment）

Scratch 3.0大部分的程式是使用JavaScript所撰寫的，但是使用到的特徵並未廣受瀏覽器所支持。基於相容性考量，在發佈與佈署之前，我們將程式碼
轉譯成ES5（JavaScript ES5版本）。任何包含於`scratch-vm` 資料夾中的擴充功能，可以使用ES6+（JavaScript ES6+版本）的特徵，但是可能要使用
`require` 去參照其他位於`scratch-vm` 資料夾的程式碼。

Most Scratch 3.0 is written using JavaScript features not yet commonly supported by browsers. For compatibility we
transpile the code to ES5 before publishing or deploying. Any extension included in the `scratch-vm` repository may
use ES6+ features and may use `require` to reference other code within the `scratch-vm` repository.

非官方的擴充功能必需要能夠自己自足。非官方擴充功能的作者必須確保瀏覽器的相容性，如果必要時亦需要協助程式轉譯的工作。

Unofficial extensions must be self-contained. Authors of unofficial extensions are responsible for ensuring browser
compatibility for those extensions, including transpiling if necessary.

## 翻譯（Translation）

Scratch擴充功能使用[ICU message format](http://userguide.icu-project.org/formatparse/messages) 進行語言間的翻譯。 有關於**核心、
團隊, 與官方**的擴充功能，可以使用`formatMessage` 的函式包裹任何的需要輸出到 [Scratch Transifex group](https://www.transifex.com/llk/public/) 翻譯的ICU messages。

Scratch extensions use the [ICU message format](http://userguide.icu-project.org/formatparse/messages) to handle
translation across languages. For **core, team, and official** extensions, the function `formatMessage` is used to
wrap any ICU messages that need to be exported to the [Scratch Transifex group](https://www.transifex.com/llk/public/)
for translation.

**所有擴充功能**也可以在`getInfo`的函式中，額外定義一個`translation_map`的物件，可以提供擴充功能本身的翻譯。以下"含有註解的範例"提供
一個較完整的範例，說明擴充功能如何處理翻譯。 **警告：** `translation_map` 的功能目前正處在提議階段，在具體實施前可能會有所變動。

**All extensions** may additionally define a `translation_map` object within the `getInfo` function which can provide
translations within an extension itself. The "Annotated Example" below provides a more complete illustration of how
translation within an extension can be managed. **WARNING:** the `translation_map` feature is currently in the
proposal phase and may change before implementation.

## 向下兼容

Scratch具有完整的版本向下兼容性。因此，程式積木與定義 *絕對不可以* 改變，這樣會造成之前儲存的專案無法載入或是無法依照預期／一致性的
方式運作。

## Backwards Compatibility

Scratch is designed to be fully backwards compatible. Because of this, block definitions and opcodes should *never*
change in a way that could cause previously saved projects to fail to load or to act in unexpected / inconsistent
ways.

## 定義擴充功能

Scratch的擴充功能被定義成一個單一的JavaScript類別，它可以參照 Scratch[VM](https://github.com/llk/scratch-vm) runtime，或是
可以處理跟Scratch VM溝通的 "代理伺服器" ，透過定義良好的工作者範圍（例如：實驗測試環境）進行溝通。

## Defining an Extension

Scratch extensions are defined as a single Javascript class which accepts either a reference to the Scratch
[VM](https://github.com/llk/scratch-vm) runtime or a "runtime proxy" which handles communication with the Scratch VM
across a well defined worker boundary (i.e. the sandbox).


以下是用JavaScript定義一個類別（SomeBlocks）的積木：

```js
class SomeBlocks {
    constructor (runtime) {
        /**
         * 儲存用以之後與 Scratch VM runtime溝通。
         * Store this for later communication with the Scratch VM runtime.
         * 如果這個擴充功能是運作在實驗測試環境，那麼 `runtime`指的是非同步的代理伺服器物件。
         * If this extension is running in a sandbox then `runtime` is an async proxy object.
         * @type {Runtime}
         */
        this.runtime = runtime;
    }

    // ...
}
```

所有的擴充功能必須定義一個命名為`getInfo`的函式，它會回傳一個物件，這個物件包含產生程式積木與擴充功能本身所需要的資訊。

All extensions must define a function called `getInfo` which returns an object that contains the information needed to
render both the blocks and the extension itself.


```js
// Core, Team, and Official extensions can `require` VM code:
// 核心、團隊與官方擴充功能可以 `require` VM程式碼：
const ArgumentType = require('../../extension-support/argument-type');
const BlockType = require('../../extension-support/block-type');

class SomeBlocks {
    // ...
    getInfo () {
        return {
            
            // 定義擴充功能的id、名稱
            id: 'someBlocks',
            name: 'Some Blocks',
            // 定義擴充功能內的積木
            blocks: [
                {
                    opcode: 'myReporter',
                    blockType: BlockType.REPORTER,
                    text: 'letter [LETTER_NUM] of [TEXT]',
                    arguments: {
                        LETTER_NUM: {
                            type: ArgumentType.STRING,
                            defaultValue: '1'
                        },
                        TEXT: {
                            type: ArgumentType.STRING,
                            defaultValue: 'text'
                        }
                    }
                }
            ]
        };
    }
    // ...
}
```

最後，擴充功能必須對於每個上面**blocks**內的"opcode"建立函式加以定義。例如：
Finally the extension must define a function for any "opcode" defined in the blocks. For example:

myReporter這個opcode在上面blocks中建立，因此需要建立一個函式說明其定義。

```js
class SomeBlocks {
    // ...
    myReporter (args) {
        return args.TEXT.charAt(args.LETTER_NUM);
    };
    // ...
}
```

### 積木參數
### Block Arguments

除了顯示文字之外，積木的參數可以是一個空格讓別的積木插入，或是下拉式選單可以從中選擇可能的數值做為參數。

In addition to displaying text, blocks can have arguments in the form of slots to take other blocks getting plugged in, or dropdown menus to select an argument value from a list of possible values.

以下為可能的積木參數：

The possible types of block arguments are as follows:

- 字串─ 字串輸入，這是一個類型的區塊，可以接受其他有回傳值的積木。
- String - a string input, this is a type-able field which also accepts other reporter blocks to be plugged in

- 數字─與之前字串的輸入類似，但是這類型的區塊只能接受數字。
- Number - an input similar to the string input, but the type-able values are constrained to numbers.

- 角度─與數字輸入類似，但是需要額外的UI（介面）以圓形指針的方式選取角度。
- Angle - an input similar to the number input, but it has an additional UI to be able to pick an angle from a
circular dial

- 布林 ─ 布林值的回傳數值積木為六角型。這個區塊並非類型區塊。
- Boolean - an input for a boolean (hexagonal shaped) reporter block. This field is not type-able.

- 顏色 ─ 選擇調色盤的顏色的輸入。這個區塊需要額外的UI（介面）選擇顏色的色調、飽和度與明亮度。有一種可能的方式是擴充功能開發者希望每次加入擴充功能加入時，顏色選擇器的預設值已經被設定好了，如果沒有預設值，擴充功能載入時會選取一個隨機的顏色。

- Color - an input which displays a color swatch. This field has additional UI to pick a color by choosing values for the color's hue, saturation and brightness. Optionally, the defaultValue for the color picker can also be chosen if the extension developer wishes to display the same color every time the extension is added. If the defaultValue is left out, the default behavior of picking a random color when the extension is loaded will be used.

-矩陣 ─ 用以輸入一個5 x 5矩陣的儲存格，每個儲存格可以是填入或是清空。
- Matrix - an input which displays a 5 x 5 matrix of cells, where each cell can be filled in or clear.

-音符 ─一種數字的輸入，用以選擇音符。這個區塊需要額外的UI（介面）從鍵盤中選擇音符。
- Note - a numeric input which can select a musical note. This field has additional UI to select a note from a
visual keyboard.

-影像─ 可以在積木上顯示影像。這是一個特別的參數類型，它並不會代表任何值，也不會接受其他積木做為參數。參見以下關於"加入影像"的章節。

- Image - an inline image displayed on a block. This is a special argument type in that it does not represent a value and does not accept other blocks to be plugged-in in place of this block field. See the section below about "Adding an Inline Image".


#### 加入影像
#### Adding an Inline Image

除了定訂積木的參數（如同上面以字串為參數的範例）時，你也可以使用影像參數。你必須包含影像的資料網址（dataURI）。如果沒有明訂，影像出現的位置會顯示空白的空間，然後在螢幕上會註記警告。

In addition to specifying block arguments (an example of string arguments shown in the code snippet above),
you can also specify an inline image for the block. You must include a dataURI for the image. If left unspecified, blank space will be allocated for the image and a warning will be logged in the console.

另外你亦可以說明`flipRTL`，它是用來設定在由右至左書寫的語文的環境下，影像是否應該水平翻轉。影像預設是不翻轉。

You can optionally also specify `flipRTL`, a property indicating whether the image should be flipped horizontally when the editor has a right to left language selected as its locale. By default, the image is not flipped.

```js
return {
    // ...
    blocks: [
        {
            //...
            arguments {
                MY_IMAGE: {
                    type: ArgumentType.IMAGE,
                    dataURI: 'myImageData',
                    alt: 'This is an image',
                    flipRTL: true
                }
            }
        }
    ]
}
```



#### 設定選單
#### Defining a Menu


在用下拉式選單顯示程式積木的參數時，設定參數中`menu`的屬性，並且在擴充功能定義中，以選單的方式來定義。
To display a drop-down menu for a block argument, specify the `menu` property of that argument and a matching item in
the `menus` section of your extension's definition:

```js
return {
    // ...
    blocks: [
        {
            // ...
            arguments: {
                FOO: {
                    type: ArgumentType.NUMBER,
                    menu: 'fooMenu'
                }
            }
        }
    ],
    menus: {
        fooMenu: {
            items: ['a', 'b', 'c']
        }
    }
}
```

選單中的項目可以使用陣列，或是回傳陣列的函式。以下為兩種最簡單選單定義的形式：
The items in a menu may be specified with an array or with the name of a function which returns an array. The two
simplest forms for menu definitions are:

```js
getInfo () {
    return {
        menus: {
            staticMenu: ['static 1', 'static 2', 'static 3'],
            dynamicMenu: 'getDynamicMenuItems'
        }
    };
}
// this member function will be called each time the menu opens
getDynamicMenuItems () {
    return ['dynamic 1', 'dynamic 2', 'dynamic 3'];
}
```

上述的例子與以下例子的有相同的定義。
The examples above are shorthand for these equivalent definitions:

```js
getInfo () {
    return {
        menus: {
            staticMenu: {
                items: ['static 1', 'static 2', 'static 3']
            },
            dynamicMenu: {
                items: 'getDynamicMenuItems'
            }
        }
    };
}
// this member function will be called each time the menu opens
getDynamicMenuItems () {
    return ['dynamic 1', 'dynamic 2', 'dynamic 3'];
}
```

如果選單項目需要一個標籤，但是標籤並等於它所對應的值-- 舉例來說，如果標籤需要顯示使用者的語言，但是它對應的值是固定的--選單項目本身可以是一個物件
而非是字串。這對於固定與動態的選單項目均適用：

If a menu item needs a label that doesn't match its value -- for example, if the label needs to be displayed in the
user's language but the value needs to stay constant -- the menu item may be an object instead of a string. This works
for both static and dynamic menu items:

```js
menus: {
    staticMenu: [
        {
            text: formatMessage(/* ... */),
            value: 42
        }
    ]
}
```

##### 以有回傳值的積木做為參數（"可以加入積木"的選單）
##### Accepting reporters ("droppable" menus)

在預設狀況下，在下拉式選單中加入一個積木做為參數是不可能的。雖然我們鼓勵擴充功能的作者儘可能能夠讓他們的選單可以加入積木做為參數，
但是這樣做需要有特別的考量以避免造成使用者的混淆與挫折感。

By default it is not possible to specify the value of a dropdown menu by inserting a reporter block. While we
encourage extension authors to make their menus accept reporters when possible, doing so requires careful
consideration to avoid confusion and frustration on the part of those using the extension.


以下為一些考量：
A few of these considerations include:

* The valid values for the menu should not change when the user changes the Scratch language setting.
  * In particular, changing languages should never break a working project.
* The average Scratch user should be able to figure out the valid values for this input without referring to extension
  documentation.
  * One way to ensure this is to make an item's text match or include the item's value. For example, the official Music
    extension contains menu items with names like "(1) Piano" with value 1, "(8) Cello" with value 8, and so on.
* The block should accept any value as input, even "invalid" values.
  * Scratch has no concept of a runtime error!
  * For a command block, sometimes the best option is to do nothing.
  * For a reporter, returning zero or the empty string might make sense.
* The block should be forgiving in its interpretation of inputs.
  * For example, if the block expects a string and receives a number it may make sense to interpret the number as a
    string instead of treating it as invalid input.

The `acceptReporters` flag indicates that the user can drop a reporter onto the menu input:

```js
menus: {
    staticMenu: {
        acceptReporters: true,
        items: [/*...*/]
    },
    dynamicMenu: {
        acceptReporters: true,
        items: 'getDynamicMenuItems'
    }
}
```

## Annotated Example

```js
// Core, Team, and Official extensions can `require` VM code:
const ArgumentType = require('../../extension-support/argument-type');
const BlockType = require('../../extension-support/block-type');
const TargetType = require('../../extension-support/target-type');

// ...or VM dependencies:
const formatMessage = require('format-message');

// Core, Team, and Official extension classes should be registered statically with the Extension Manager.
// See: scratch-vm/src/extension-support/extension-manager.js
class SomeBlocks {
    constructor (runtime) {
        /**
         * Store this for later communication with the Scratch VM runtime.
         * If this extension is running in a sandbox then `runtime` is an async proxy object.
         * @type {Runtime}
         */
        this.runtime = runtime;
    }

    /**
     * @return {object} This extension's metadata.
     */
    getInfo () {
        return {
            // Required: the machine-readable name of this extension.
            // Will be used as the extension's namespace.
            id: 'someBlocks',

            // Core extensions only: override the default extension block colors.
            color1: '#FF8C1A',
            color2: '#DB6E00',

            // Optional: the human-readable name of this extension as string.
            // This and any other string to be displayed in the Scratch UI may either be
            // a string or a call to `formatMessage`; a plain string will not be
            // translated whereas a call to `formatMessage` will connect the string
            // to the translation map (see below). The `formatMessage` call is
            // similar to `formatMessage` from `react-intl` in form, but will actually
            // call some extension support code to do its magic. For example, we will
            // internally namespace the messages such that two extensions could have
            // messages with the same ID without colliding.
            // See also: https://github.com/yahoo/react-intl/wiki/API#formatmessage
            name: formatMessage({
                id: 'extensionName',
                defaultMessage: 'Some Blocks',
                description: 'The name of the "Some Blocks" extension'
            }),

            // Optional: URI for a block icon, to display at the edge of each block for this
            // extension. Data URI OK.
            // TODO: what file types are OK? All web images? Just PNG?
            blockIconURI: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAkAAAAFCAAAAACyOJm3AAAAFklEQVQYV2P4DwMMEMgAI/+DEUIMBgAEWB7i7uidhAAAAABJRU5ErkJggg==',

            // Optional: URI for an icon to be displayed in the blocks category menu.
            // If not present, the menu will display the block icon, if one is present.
            // Otherwise, the category menu shows its default filled circle.
            // Data URI OK.
            // TODO: what file types are OK? All web images? Just PNG?
            menuIconURI: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAkAAAAFCAAAAACyOJm3AAAAFklEQVQYV2P4DwMMEMgAI/+DEUIMBgAEWB7i7uidhAAAAABJRU5ErkJggg==',

            // Optional: Link to documentation content for this extension.
            // If not present, offer no link.
            docsURI: 'https://....',

            // Required: the list of blocks implemented by this extension,
            // in the order intended for display.
            blocks: [
                {
                    // Required: the machine-readable name of this operation.
                    // This will appear in project JSON.
                    opcode: 'myReporter', // becomes 'someBlocks.myReporter'

                    // Required: the kind of block we're defining, from a predefined list.
                    // Fully supported block types:
                    //   BlockType.BOOLEAN - same as REPORTER but returns a Boolean value
                    //   BlockType.COMMAND - a normal command block, like "move {} steps"
                    //   BlockType.HAT - starts a stack if its value changes from falsy to truthy ("edge triggered")
                    //   BlockType.REPORTER - returns a value, like "direction"
                    // Block types in development or for internal use only:
                    //   BlockType.BUTTON - place a button in the block palette
                    //   BlockType.CONDITIONAL - control flow, like "if {}" or "if {} else {}"
                    //     A CONDITIONAL block may return the one-based index of a branch to
                    //     run, or it may return zero/falsy to run no branch.
                    //   BlockType.EVENT - starts a stack in response to an event (full spec TBD)
                    //   BlockType.LOOP - control flow, like "repeat {} {}" or "forever {}"
                    //     A LOOP block is like a CONDITIONAL block with two differences:
                    //     - the block is assumed to have exactly one child branch, and
                    //     - each time a child branch finishes, the loop block is called again.
                    blockType: BlockType.REPORTER,

                    // Required for CONDITIONAL blocks, ignored for others: the number of
                    // child branches this block controls. An "if" or "repeat" block would
                    // specify a branch count of 1; an "if-else" block would specify a
                    // branch count of 2.
                    // TODO: should we support dynamic branch count for "switch"-likes?
                    branchCount: 0,

                    // Optional, default false: whether or not this block ends a stack.
                    // The "forever" and "stop all" blocks would specify true here.
                    terminal: true,

                    // Optional, default false: whether or not to block all threads while
                    // this block is busy. This is for things like the "touching color"
                    // block in compatibility mode, and is only needed if the VM runs in a
                    // worker. We might even consider omitting it from extension docs...
                    blockAllThreads: false,

                    // Required: the human-readable text on this block, including argument
                    // placeholders. Argument placeholders should be in [MACRO_CASE] and
                    // must be [ENCLOSED_WITHIN_SQUARE_BRACKETS].
                    text: formatMessage({
                        id: 'myReporter',
                        defaultMessage: 'letter [LETTER_NUM] of [TEXT]',
                        description: 'Label on the "myReporter" block'
                    }),

                    // Required: describe each argument.
                    // Argument order may change during translation, so arguments are
                    // identified by their placeholder name. In those situations where
                    // arguments must be ordered or assigned an ordinal, such as interaction
                    // with Scratch Blocks, arguments are ordered as they are in the default
                    // translation (probably English).
                    arguments: {
                        // Required: the ID of the argument, which will be the name in the
                        // args object passed to the implementation function.
                        LETTER_NUM: {
                            // Required: type of the argument / shape of the block input
                            type: ArgumentType.NUMBER,

                            // Optional: the default value of the argument
                            default: 1
                        },

                        // Required: the ID of the argument, which will be the name in the
                        // args object passed to the implementation function.
                        TEXT: {
                            // Required: type of the argument / shape of the block input
                            type: ArgumentType.STRING,

                                // Optional: the default value of the argument
                            default: formatMessage({
                                id: 'myReporter.TEXT_default',
                                defaultMessage: 'text',
                                description: 'Default for "TEXT" argument of "someBlocks.myReporter"'
                            })
                        }
                    },

                    // Optional: the function implementing this block.
                    // If absent, assume `func` is the same as `opcode`.
                    func: 'myReporter',

                    // Optional: list of target types for which this block should appear.
                    // If absent, assume it applies to all builtin targets -- that is:
                    // [TargetType.SPRITE, TargetType.STAGE]
                    filter: [TargetType.SPRITE]
                },
                {
                    // Another block...
                }
            ],

            // Optional: define extension-specific menus here.
            menus: {
                // Required: an identifier for this menu, unique within this extension.
                menuA: [
                    // Static menu: list items which should appear in the menu.
                    {
                        // Required: the value of the menu item when it is chosen.
                        value: 'itemId1',

                        // Optional: the human-readable label for this item.
                        // Use `value` as the text if this is absent.
                        text: formatMessage({
                            id: 'menuA_item1',
                            defaultMessage: 'Item One',
                            description: 'Label for item 1 of menu A in "Some Blocks" extension'
                        })
                    },

                    // The simplest form of a list item is a string which will be used as
                    // both value and text.
                    'itemId2'
                ],

                // Dynamic menu: returns an array as above.
                // Called each time the menu is opened.
                menuB: 'getItemsForMenuB',

                // The examples above are shorthand for setting only the `items` property in this full form:
                menuC: {
                    // This flag makes a "droppable" menu: the menu will allow dropping a reporter in for the input.
                    acceptReporters: true,

                    // The `item` property may be an array or function name as in previous menu examples.
                    items: [/*...*/] || 'getItemsForMenuC'
                }
            },

            // Optional: translations (UNSTABLE - NOT YET SUPPORTED)
            translation_map: {
                de: {
                    'extensionName': 'Einige Blöcke',
                    'myReporter': 'Buchstabe [LETTER_NUM] von [TEXT]',
                    'myReporter.TEXT_default': 'Text',
                    'menuA_item1': 'Artikel eins',

                    // Dynamic menus can be translated too
                    'menuB_example': 'Beispiel',

                    // This message contains ICU placeholders (see `myReporter()` below)
                    'myReporter.result': 'Buchstabe {LETTER_NUM} von {TEXT} ist {LETTER}.'
                },
                it: {
                    // ...
                }
            }
        };
    };

    /**
     * Implement myReporter.
     * @param {object} args - the block's arguments.
     * @property {string} MY_ARG - the string value of the argument.
     * @returns {string} a string which includes the block argument value.
     */
    myReporter (args) {
        // This message contains ICU placeholders, not Scratch placeholders
        const message = formatMessage({
            id: 'myReporter.result',
            defaultMessage: 'Letter {LETTER_NUM} of {TEXT} is {LETTER}.',
            description: 'The text template for the "myReporter" block result'
        });

        // Note: this implementation is not Unicode-clean; it's just here as an example.
        const result = args.TEXT.charAt(args.LETTER_NUM);

        return message.format({
            LETTER_NUM: args.LETTER_NUM,
            TEXT: args.TEXT,
            LETTER: result
        });
    };
}
```
