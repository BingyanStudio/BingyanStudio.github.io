---
title: 浅谈 markdown-it 原理
cover_image: /img/cover.png
abbrlink: 9cddee5
date: 2023-11-02 13:53:10
---

markdown 是一种轻量级标记语言，使用易读易写的纯文本格式编写文档，排版语法简洁。markdown-it 是一个由 javascript 语言编写的 markdown 解析器。通过对 markdown-it 源码的学习，我们可以大致了解到 markdown 语法是如何被解析成 html 语句的。

## markdown 解析的整体流程

markdown-it 通过 parse 函数将字符串转换为 token ，在 render 函数通过 token 生成我们需要的 html 字符串。

可以在 https://markdown-it.github.io/ 中点击右上角 debug 按钮查看通过 parse 得到的 token。

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Q41vPhOcHwdRcfkwER3z2jMZ9BOLCXsJ8ziccyfRTvXTPTnumug9SqThicYTYBQibCm86ib3ppKFomAA/640?wx_fmt=png)markdown解析流程## markdown-it 入口文件

通过 markdown-it 的入口文件，我们可以了解到 markdown-it 的基本功能。

在 markdown-it/lib/index.js 文件中我们可以看到

* MarkdownIt 类的初始化

function MarkdownIt(presetName, options)     // 初始化preset  if (!options) ;      presetName = 'default';    }  }      // parse、tokenize、render流程中主要使用的类  this.inline = new ParserInline();   this.block = new ParserBlock();  this.core = new ParserCore();  this.renderer = new Renderer();  // 主要用于校验 url 的合法以及 decode 与 encode    this.linkify = new LinkifyIt();  this.validateLink = validateLink;  this.normalizeLink = normalizeLink;  this.normalizeLinkText = normalizeLinkText;  this.utils = utils;  this.helpers = utils.assign({}, helpers);  this.options = {};  this.configure(presetName);  if (options) }
* 为 MarkdownIt 类定义方法

// 合并optionsMarkdownIt.prototype.set = function (options) ;// 根据presets禁用 ParserInline、ParserBlock、ParserCore 的某些规则MarkdownIt.prototype.configure = function (presets) ;// 开启 ParserInline、ParserBlock、ParserCore 的某些规则MarkdownIt.prototype.enable = function (list, ignoreInvalid) ;// 关闭 ParserInline、ParserBlock、ParserCore 的某些规则MarkdownIt.prototype.disable = function (list, ignoreInvalid) ;// 使用插件MarkdownIt.prototype.use = function (plugin /*, params, ... */) ;// parse入口MarkdownIt.prototype.parse = function (src, env)   var state = new this.core.State(src, this, env);  this.core.process(state);  return state.tokens;};// render入口MarkdownIt.prototype.render = function (src, env) ;  return this.renderer.render(this.parse(src, env), this.options, env);};// 仅用于编译inline类型tokenMarkdownIt.prototype.parseInline = function (src, env) ;// 接收parseInline的token并生成htmlMarkdownIt.prototype.renderInline = function (src, env) ;  return this.renderer.render(this.parseInline(src, env), this.options, env);};
## ParserCore

parse 函数中调用了 ParserCore 类的 process 方法获得 token。我们可以在 lib/parser_core.js 中看到 ParserCore 的逻辑。

```
function Core() }
```
#### Ruler

ParserCore 类中只有 ruler 一个变量，想要更好的了解 ParserCore 的功能，我们首先要认识一下 Ruler 类。

```
function Ruler()   //  this.__rules__ = [];    // 存放职责链的信息  this.__cache__ = null;}
```
在 Ruler 类中会储存很多 rule 处理函数。在 parse 的过程中，Ruler 负责调用 parse 相关的 rule 处理函数生成 token

#### process 过程

在 MarkdownIt.prototype.parse 中执行了 this.core.process(state)。以下是 process 的相关代码

```
// parser_core.jsvar _rules = [  [ 'normalize',      require('./rules_core/normalize')      ],  [ 'block',          require('./rules_core/block')          ],  [ 'inline',         require('./rules_core/inline')         ],  [ 'linkify',        require('./rules_core/linkify')        ],  [ 'replacements',   require('./rules_core/replacements')   ],  [ 'smartquotes',    require('./rules_core/smartquotes')    ],  [ 'text_join',      require('./rules_core/text_join')      ]];function Core() }// 按顺序执行_rules中的rule处理函数Core.prototype.process = function (state) };// ruler.jsRuler.prototype.push = function (ruleName, fn, options) ;      this.__rules__.push();  this.__cache__ = null;};Ruler.prototype.getRules = function (chainName)     this.__compile__();  }      return this.__cache__[chainName] || [];};
```
Core 构造函数将 _rules 数组通过 push 方法储存在 Ruler 类中，可以通过 getRules 获取所需职责链上的 rule 处理函数。

process 获取了 _rules 中所有元素，并按顺序执行。其中，parse 过程最核心的部分在 block 和 inline 中，分别对应着 ParserBlock 和 ParserInline 两个类。

## token

我们先来了解一下 ParseBlock 与 ParserInline 需要生成怎样的 token，在 lib/token.js 中可以看到 Token 类的定义

```
function Token(type, tag, nesting) 
```
## ParserBlock

在 lib/parser_block 中可以看到 ParserBlock 的定义

```
function ParserBlock() );  }}
```
类似于 ParserCore ，ParserBlock 首先导入生成 block token 所需要的 rule。

```
ParserBlock.prototype.parse = function (src, md, env, outTokens)   state = new this.State(src, md, env, outTokens);  this.tokenize(state, state.line, state.lineMax);};ParserBlock.prototype.State = require('./rules_block/state_block');
```
在 tokenize 之前，parse 方法首先创建了一个 state 用于管理 tokenize 过程中的一些状态。在 lib/rules_block/state_block.js 中可以看到 StateBlock 类的定义。

```
function StateBlock(src, md, env, tokens) 
```
接下来看看 tokenize 函数是如何生成 token 的

```
ParserBlock.prototype.tokenize = function (state, startLine, endLine)     if (state.sCount[line] < state.blkIndent)     if (state.level >= maxNesting)     prevLine = state.line;    // 按顺序执行rule处理函数生成token    for (i = 0; i < len; i++)         break;      }    }    if (!ok) throw new Error('none of the block rules matched');     state.tight = !hasEmptyLines;    if (state.isEmpty(state.line - 1))     line = state.line;    if (line < endLine && state.isEmpty(line))   }};
```
生成 token 最主要的步骤就是 ok = rules[i](state, line, endLine, false); 这一步。通过顺序执行 rule 处理函数对每一种 markdown 语法进行逐一判断，如果符合则更新 token 并返回 true 。

```
var _rules = [  [ 'table',      require('./rules_block/table'),      [ 'paragraph', 'reference' ] ],  [ 'code',       require('./rules_block/code') ],  [ 'fence',      require('./rules_block/fence'),      [ 'paragraph', 'reference', 'blockquote', 'list' ] ],  [ 'blockquote', require('./rules_block/blockquote'), [ 'paragraph', 'reference', 'blockquote', 'list' ] ],  [ 'hr',         require('./rules_block/hr'),         [ 'paragraph', 'reference', 'blockquote', 'list' ] ],  [ 'list',       require('./rules_block/list'),       [ 'paragraph', 'reference', 'blockquote' ] ],  [ 'reference',  require('./rules_block/reference') ],  [ 'html_block', require('./rules_block/html_block'), [ 'paragraph', 'reference', 'blockquote' ] ],  [ 'heading',    require('./rules_block/heading'),    [ 'paragraph', 'reference', 'blockquote' ] ],  [ 'lheading',   require('./rules_block/lheading') ],  [ 'paragraph',  require('./rules_block/paragraph') ]];
```
块级元素的 rule 有以上这些，基本上都能根据名称判断他们各自负责生成的 token 类型。以比较常用的标题标签（#、##类语法 ）为例：

```
module.exports = function heading(state, startLine, endLine, silent)   // 判断是否为code block  ch  = state.src.charCodeAt(pos);  // 获取当前位置字符的Unicode  if (ch !== 0x23/* # */ || pos >= max)   // 当前字符不为#则说明不符合标题语法  // 计算标题等级（#的数量）  level = 1;  ch = state.src.charCodeAt(++pos);  while (ch === 0x23/* # */ && pos < max && level <= 6)   if (level > 6 || (pos < max && !isSpace(ch)))   if (silent)   // 删除字符串末尾形如'    ###  '的字符串  max = state.skipSpacesBack(max, pos);  tmp = state.skipCharsBack(max, 0x23, pos); // #  if (tmp > pos && isSpace(state.src.charCodeAt(tmp - 1)))     // 进入下一行    state.line = startLine + 1;    // 生成标题语法的token    token        = state.push('heading_open', 'h' + String(level), 1);  token.markup = '########'.slice(0, level);  token.map    = [ startLine, state.line ];  token          = state.push('inline', '', 0);  token.content  = state.src.slice(pos, max).trim();  token.map      = [ startLine, state.line ];  token.children = [];  token        = state.push('heading_close', 'h' + String(level), -1);  token.markup = '########'.slice(0, level);  return true;};
```
再来看看比较常用的代码块：

```
module.exports = function fence(state, startLine, endLine, silent)   // 判断是否为code block  if (pos + 3 > max)   // 如果这一行没有三个字符，则肯定不是```语法  marker = state.src.charCodeAt(pos);  if (marker !== 0x7E/* ~ */ && marker !== 0x60 /* ` */)   mem = pos;  pos = state.skipChars(pos, marker); // 跳过相同的`字符  len = pos - mem;  if (len < 3)   markup = state.src.slice(mem, pos);  params = state.src.slice(pos, max);  if (params.indexOf(String.fromCharCode(marker)) >= 0)   // 如果这是结尾的```， 则可以直接返回  if (silent)   // 寻找结尾的```语法  nextLine = startLine;  for (;;)     pos = mem = state.bMarks[nextLine] + state.tShift[nextLine];    max = state.eMarks[nextLine];    if (pos < max && state.sCount[nextLine] < state.blkIndent)     if (state.src.charCodeAt(pos) !== marker)     if (state.sCount[nextLine] - state.blkIndent >= 4)     pos = state.skipChars(pos, marker);    if (pos - mem < len)   // 结尾的`数量应不少于开始的数量    pos = state.skipSpaces(pos);  // 确保末尾只有空格    if (pos < max)     haveEndMarker = true;        break;  }  len = state.sCount[startLine];  state.line = nextLine + (haveEndMarker ? 1 : 0);    // 生成fence token  token         = state.push('fence', 'code', 0);  token.info    = params;  token.content = state.getLines(startLine + 1, nextLine, len, true);  token.markup  = markup;  token.map     = [ startLine, state.line ];  return true;};
```
## ParserInline

在 parserBlock 之后，token 中常常会出现类似 content 为 __ad__ 的，加粗语法尚未解析的元素，这个时候就需要 Parser Inline 进行进一步的解析

```
module.exports = function inline(state)   }};
```
同样的，我们先来看看 inline 中的 state 储存了哪些属性

```
function StateInline(src, md, env, outTokens) ;  this.delimiters = [];  // 存放一些特殊标记的分隔符，比如*、~等}
```
在 lib/parser_inline.js 中我们可以看到 ParserInline 的定义

```
var _rules = [  [ 'text',            require('./rules_inline/text') ],  // 提取连续的非 isTerminatorChar 字符  [ 'newline',         require('./rules_inline/newline') ],  // 处理换行符 \n  [ 'escape',          require('./rules_inline/escape') ],  // 处理转义字符 \  [ 'backticks',       require('./rules_inline/backticks') ],  // 处理反引号字符 `  [ 'strikethrough',   require('./rules_inline/strikethrough').tokenize ], // 处理删除字符 ~  [ 'emphasis',        require('./rules_inline/emphasis').tokenize ],  // 处理加粗文字的字符 *或者_  [ 'link',            require('./rules_inline/link') ],  // 解析超链接  [ 'image',           require('./rules_inline/image') ],  // 解析图片  [ 'autolink',        require('./rules_inline/autolink') ],  // 解析 < 与 > 之间的 url  [ 'html_inline',     require('./rules_inline/html_inline') ],  // 解析HTML行内标签  [ 'entity',          require('./rules_inline/entity') ]  // 解析HTML实体标签，比如 、"、'等等];var _rules2 = [  [ 'balance_pairs',   require('./rules_inline/balance_pairs') ], // 给诸如*、~等找到配对的开闭标签  [ 'strikethrough',   require('./rules_inline/strikethrough').postProcess ],  // 处理~字符，生成标签的token  [ 'emphasis',        require('./rules_inline/emphasis').postProcess ],  // 处理*或者_字符，生成或者标签的token  [ 'text_collapse',   require('./rules_inline/text_collapse') ]  // 合并相邻的文本节点];function ParserInline()   this.ruler2 = new Ruler();  for (i = 0; i < _rules2.length; i++) }
```
与 ParserBlock 不同的是，ParserInline 有两个 rule 实例，一个在 tokenize 时调用，一个在 tokenize 后调用

```
ParserInline.prototype.tokenize = function (state)       }    }    if (ok)       continue;    }    state.pending += state.src[state.pos++];  }  if (state.pending) };ParserInline.prototype.parse = function (str, md, env, outTokens) };
```
## Render

render 函数根据 token 的 type 进行渲染

```
Renderer.prototype.render = function (tokens, options, env)  else if (typeof rules[type] !== 'undefined')  else   }  return result;};
```
* 当 type 为 inline 时
```
Renderer.prototype.renderInline = function (tokens, options, env)  else   }  return result;};
```
* 当 type 为 undefined 时
```
Renderer.prototype.renderToken = function renderToken(tokens, idx, options)   if (token.block && token.nesting !== -1 && idx && tokens[idx - 1].hidden)     // 添加元素名称，如  result += (token.nesting === -1 ? ' : '<') + token.tag;  // 添加元素属性，如  result += this.renderAttrs(token);  // 添加元素结束符，如  if (token.nesting === 0 && options.xhtmlOut)   // 判断是否需要在末尾添加换行    if (token.block)  else if (nextToken.nesting === -1 && nextToken.tag === token.tag)       }    }  }  result += needLf ? '>\n' : '>';  return result;};
```

