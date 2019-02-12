<template>
    <div class='outer-box'>
        <div class="confirm-sign" :style="{opacity: error ? 0 : 1}">
            <svg height='20px'
                 width='20px'
                 viewBox='0 0 100 100'>
                <path fillRule='evenodd'
                      clipRule='evenodd'
                      fill='green'
                      opacity='0.85'
                      d='M39.363,79L16,55.49l11.347-11.419L39.694,56.49L72.983,23L84,34.085L39.363,79z'
                ></path>
            </svg>
        </div>
        <div class="container">
            <div class='warning-box' v-if="error" :style="{backgroundColor: colors.background_warning}">
                <div class="warning-sign-part">
                    <div class="warning-sign-box">
                        <div class="warning-sign">
                            <svg height='20px'
                                 width='20px'
                                 viewBox='0 0 100 100'>
                                <path fillRule='evenodd' clipRule='evenodd'
                                      fill='red'
                                      d='M73.9,5.75c0.467-0.467,1.067-0.7,1.8-0.7c0.7,0,1.283,0.233,1.75,0.7l16.8,16.8  c0.467,0.5,0.7,1.084,0.7,1.75c0,0.733-0.233,1.334-0.7,1.801L70.35,50l23.9,23.95c0.5,0.467,0.75,1.066,0.75,1.8  c0,0.667-0.25,1.25-0.75,1.75l-16.8,16.75c-0.534,0.467-1.117,0.7-1.75,0.7s-1.233-0.233-1.8-0.7L50,70.351L26.1,94.25  c-0.567,0.467-1.167,0.7-1.8,0.7c-0.667,0-1.283-0.233-1.85-0.7L5.75,77.5C5.25,77,5,76.417,5,75.75c0-0.733,0.25-1.333,0.75-1.8  L29.65,50L5.75,26.101C5.25,25.667,5,25.066,5,24.3c0-0.666,0.25-1.25,0.75-1.75l16.8-16.8c0.467-0.467,1.05-0.7,1.75-0.7  c0.733,0,1.333,0.233,1.8,0.7L50,29.65L73.9,5.75z'></path>
                            </svg>
                        </div>
                    </div>
                </div>
                <span class="warning-message-part">
                    <p class="warning-message">
                        {{renderErrorMessage()}}
                    </p>
                </span>
            </div>
            <div name="body"
                class="main-body"
                :style="{height: error ? 'calc( 100% - 60px )' : '100%'}">
                <span name="labels"
                      ref="refLabels"
                      class="main-body-label">
                        <div v-for="number in labels"
                             :style="{color: error.line === number ? 'red': colors.default}"
                             :key="number + Math.random()">{{number}}</div>
                </span>
                <span class="main-body-content"
                      ref="refContent"
                      :contentEditable="true"
                      v-html="markup"
                      autoComplete='off'
                      autoCorrect='off'
                      autoCapitalize='off'
                      @keydown="onKeyDown"
                      @click="onClick"
                      @blur="onBlur"
                      @paste="onPaste"
                      @keypress="onKeyPress"
                      @scroll="onScroll"
                ></span>
            </div>
        </div>
    </div>
</template>

<script>
    import defaultLocale from './en';
    import themes from './themes';
    import mitsuketa from './mitsuketa/index';
    import {format} from './locale/index';

    export default {
        data() {
            return {
                colors: themes.light_mitsuketa_tribute, // 颜色主题
                labels: 0, // 行数
                error: false,
                markup: '' // json html
            };
        },
        props: {
            viewOnly: false,
            value: [String, Object, Array]
        },
        watch: {
            value(newValue, oldValue) {
                const shouldUpdate = mitsuketa.identical(newValue, oldValue);
                if (shouldUpdate) this.initData(newValue);
            }
        },
        methods: {
            initData(v) {
                if ([null, undefined, ''].indexOf(v) > -1) return;

                let jsonData;
                const dataType = mitsuketa.getType(v);

                if (['object', 'array'].indexOf(dataType) > -1) {
                    jsonData = JSON.parse(JSON.stringify(v))
                } else if (dataType === 'string') {
                    try {
                        jsonData = JSON.parse(v);
                    } catch (e) {
                        this.markup = '';
                        this.labels = 0;
                        throw Error('expects json string properties only. can not parse json.');
                    }
                } else {
                    this.markup = '';
                    this.labels = 0;
                    throw Error('expects object/string properties only. Got \'' + typeof v + '\' type instead.');
                }
                const data = this.tokenize(jsonData);
                this.markup = data.markup;
                this.labels = data.lines - 1;
            },
            onKeyPress(event) {
                this.stopEvent(event);
                if (this.viewOnly) return;
                this.setUpdateTime();
            },
            onClick(event) {
                this.stopEvent(event);
                if (this.viewOnly) return;
            },
            onBlur(event) {
                this.stopEvent(event);
                if (this.viewOnly) return;
                this.setUpdateTime();
            },
            onScroll(event) {
                this.$refs.refLabels.scrollTop = event.target.scrollTop;
            },
            onPaste(event) {
                if (this.viewOnly) {
                    this.stopEvent(event);
                    return;
                }
                event.preventDefault();
                var text = event.clipboardData.getData('text/plain');
                document.execCommand('insertHTML', false, text);
                this.update();
            },
            onKeyDown(event) {
                switch (event.key) {
                    case 'Tab':
                        this.stopEvent(event);
                        if (this.viewOnly) break;
                        document.execCommand('insertText', false, '  ');
                        this.setUpdateTime();
                        break;
                    case 'Backspace' :
                    case 'Delete'     :
                        if (this.viewOnly) this.stopEvent();
                        this.setUpdateTime();
                        break;
                    case 'ArrowLeft' :
                    case 'ArrowRight' :
                    case 'ArrowUp'   :
                    case 'ArrowDown'  :
                        this.setUpdateTime();
                        break;
                    case 'a'         :
                    case 'c'          :
                        if (this.viewOnly) this.stopEvent();
                        break;
                    default :
                        if (this.viewOnly) this.stopEvent();
                        break;
                }
            },
            stopEvent(event) {
                if (!event) return;
                event.preventDefault();
                event.stopPropagation();
            },
            newSpan(i, token, depth) {
                let
                    colors = this.colors,
                    type = token.type,
                    string = token.string;
                let color = '';
                switch (type) {
                    case 'string' :
                    case 'number' :
                    case 'primitive' :
                    case 'error' :
                        color = colors[token.type];
                        break;
                    case 'key'    :
                        if (string === ' ') color = colors.keys_whiteSpace; else color = colors.keys;
                        break;
                    case 'symbol' :
                        if (string === ':') color = colors.colon; else color = colors.default;
                        break;
                    default :
                        color = colors.default;
                        break;
                }
                if (string.length !== string.replace(/</g, '').replace(/>/g, '').length) string = '<xmp style=display:inline;>' + string + '</xmp>';
                return (
                    '<span' +
                    ' type="' + type + '"' +
                    ' value="' + string + '"' +
                    ' depth="' + depth + '"' +
                    ' style="color:' + color + '"' +
                    '>' + string +
                    '</span>'
                );
            },
            getCursorPosition(countBR) {
                /**
                 * Need to deprecate countBR
                 * It is used to differenciate between good markup render, and aux render when error found
                 * Adjustments based on coundBR account for usage of <br> instead of <span> for linebreaks to determine acurate cursor position
                 * Find a way to consolidate render styles
                 */
                const isChildOf = node => {
                    while (node !== null) {
                        if (node === this.$refs.refContent) return true;
                        node = node.parentNode;
                    }
                    return false;
                };
                let
                    selection = window.getSelection(),
                    charCount = -1,
                    linebreakCount = 0,
                    node;
                if (selection.focusNode && isChildOf(selection.focusNode)) {
                    node = selection.focusNode;
                    charCount = selection.focusOffset;
                    while (node) {
                        if (node === this.$refs.refContent) break;
                        if (node.previousSibling) {
                            node = node.previousSibling;
                            if (countBR) if (node.nodeName === 'BR') linebreakCount++;
                            charCount += node.textContent.length;
                        } else {
                            node = node.parentNode;
                            if (node === null) break;
                        }
                    }
                }

                return charCount + linebreakCount;
            },
            setCursorPosition(nextPosition) {
                if ([false, null, undefined].indexOf(nextPosition) > -1) return;
                const createRange = (node, chars, range) => {
                    if (!range) {
                        range = document.createRange();
                        range.selectNode(node);
                        range.setStart(node, 0);
                    }
                    if (chars.count === 0) {
                        range.setEnd(node, chars.count);
                    } else if (node && chars.count > 0) {
                        if (node.nodeType === Node.TEXT_NODE) {
                            if (node.textContent.length < chars.count) chars.count -= node.textContent.length;
                            else {
                                range.setEnd(node, chars.count);
                                chars.count = 0;
                            }
                        } else
                            for (var lp = 0; lp < node.childNodes.length; lp++) {
                                range = createRange(node.childNodes[lp], chars, range);
                                if (chars.count === 0) break;
                            }
                    }
                    return range;
                };
                const setPosition = chars => {
                    if (chars < 0) return;
                    let
                        selection = window.getSelection(),
                        range = createRange(this.$refs.refContent, {count: chars});
                    if (!range) return;
                    range.collapse(false);
                    selection.removeAllRanges();
                    selection.addRange(range);
                };
                if (nextPosition > 0) setPosition(nextPosition);
                else this.$refs.refContent.focus();
            },
            scheduledUpdate() {
                const {updateTime} = this;
                if (updateTime === false) return;
                if (updateTime > new Date().getTime()) return;
                this.update();
            },
            setUpdateTime() {
                this.updateTime = new Date().getTime() + 1000;
            },
            update(cursorOffset = 0) {
                this.updateTime = false;

                const container = this.$refs.refContent;
                const data = this.tokenize(container);

                this.error = data.error;
                this.markup = data.markup;
                if (!this.error) {
                    this.$emit('input', data.json);
                }

                let cursorPosition = this.getCursorPosition(data.error) + cursorOffset;

                this.$nextTick(() => {
                    this.setCursorPosition(cursorPosition);
                });

                this.labels = data.lines - 1;

            },

            // 错误信息
            renderErrorMessage() {
                return format(defaultLocale.format, this.error);

            },
            tokenize(something) {
                if (typeof something !== 'object') return console.error('tokenize() expects object type properties only. Got \'' + typeof something + '\' type instead.');
                const locale = defaultLocale;
                const newSpan = this.newSpan;
                /**
                 *     DOM NODE || ONBLUR OR UPDATE
                 */

                if ('nodeType' in something) {
                    const
                        containerNode = something.cloneNode(true),
                        hasChildren = containerNode.hasChildNodes();
                    if (!hasChildren) return '';
                    const children = containerNode.childNodes;

                    let buffer = {
                        tokens_unknown: [],
                        tokens_proto: [],
                        tokens_split: [],
                        tokens_fallback: [],
                        tokens_normalize: [],
                        tokens_merge: [],
                        tokens_plainText: '',
                        indented: '',
                        json: '',
                        jsObject: undefined,
                        markup: ''
                    };
                    for (var i = 0; i < children.length; i++) {
                        let child = children[i];
                        let info = {};
                        switch (child.nodeName) {
                            case 'SPAN' :
                                info = {
                                    string: child.textContent,
                                    type: child.attributes.type.textContent
                                };
                                buffer.tokens_unknown.push(info);
                                break;
                            case 'DIV' :
                                buffer.tokens_unknown.push({string: child.textContent, type: 'unknown'});
                                break;
                            case 'BR' :
                                if (child.textContent === '')
                                    buffer.tokens_unknown.push({string: '\n', type: 'unknown'});
                                break;
                            case '#text' :
                                buffer.tokens_unknown.push({string: child.wholeText, type: 'unknown'});
                                break;
                            case 'FONT' :
                                buffer.tokens_unknown.push({string: child.textContent, type: 'unknown'});
                                break;
                            default :
                                console.error('Unrecognized node:', {child});
                                break;
                        }
                    }

                    function quarkize(text, prefix = '') {
                        let
                            buffer = {
                                active: false,
                                string: '',
                                number: '',
                                symbol: '',
                                space: '',
                                delimiter: '',
                                quarks: []
                            };

                        function pushAndStore(char, type) {
                            switch (type) {
                                case 'symbol' :
                                case 'delimiter' :
                                    if (buffer.active) buffer.quarks.push({
                                        string: buffer[buffer.active],
                                        type: prefix + '-' + buffer.active
                                    });
                                    buffer[buffer.active] = '';
                                    buffer.active = type;
                                    buffer[buffer.active] = char;
                                    break;
                                default :
                                    if (type !== buffer.active || ([buffer.string, char].indexOf('\n') > -1)) {
                                        if (buffer.active) buffer.quarks.push({
                                            string: buffer[buffer.active],
                                            type: prefix + '-' + buffer.active
                                        });
                                        buffer[buffer.active] = '';
                                        buffer.active = type;
                                        buffer[buffer.active] = char;
                                    }
                                    else buffer[type] += char;
                                    break;
                            }
                        }

                        function finalPush() {
                            if (buffer.active) {
                                buffer.quarks.push({
                                    string: buffer[buffer.active],
                                    type: prefix + '-' + buffer.active
                                });
                                buffer[buffer.active] = '';
                                buffer.active = false;
                            }
                        }

                        for (var i = 0; i < text.length; i++) {
                            const char = text.charAt(i);
                            switch (char) {
                                case '"'      :
                                case '\''      :
                                    pushAndStore(char, 'delimiter');
                                    break;
                                case ' '      :
                                case '\u00A0' :
                                    pushAndStore(char, 'space');
                                    break;
                                case '{'      :
                                case '}'      :
                                case '['      :
                                case ']'      :
                                case ':'      :
                                case ','      :
                                    pushAndStore(char, 'symbol');
                                    break;
                                case '0'      :
                                case '1'      :
                                case '2'      :
                                case '3'      :
                                case '4'      :
                                case '5'      :
                                case '6'      :
                                case '7'      :
                                case '8'      :
                                case '9'      :
                                    if (buffer.active === 'string') pushAndStore(char, 'string');
                                    else pushAndStore(char, 'number');
                                    break;
                                case '-'  :
                                    if (i < text.length - 1)
                                        if ('0123456789'.indexOf(text.charAt(i + 1)) > -1) {
                                            pushAndStore(char, 'number');
                                            break;
                                        }
                                case '.' :
                                    if (i < text.length - 1 && i > 0)
                                        if (
                                            '0123456789'.indexOf(text.charAt(i + 1)) > -1 &&
                                            '0123456789'.indexOf(text.charAt(i - 1)) > -1
                                        ) {
                                            pushAndStore(char, 'number');
                                            break;
                                        }
                                default :
                                    pushAndStore(char, 'string');
                                    break;
                            }
                        }
                        finalPush();
                        return buffer.quarks;
                    }

                    for (var i = 0; i < buffer.tokens_unknown.length; i++) {
                        let token = buffer.tokens_unknown[i];
                        buffer.tokens_proto = buffer.tokens_proto.concat(quarkize(token.string, 'proto'));
                    }

                    function validToken(string, type) {
                        const quotes = '\'"';
                        let
                            firstChar = '',
                            lastChar = '',
                            quoteType = false;
                        switch (type) {
                            case 'primitive' :
                                if (['true', 'false', 'null', 'undefined'].indexOf(string) === -1) return false;
                                break;
                            case 'string' :
                                if (string.length < 2) return false;
                                firstChar = string.charAt(0),
                                    lastChar = string.charAt(string.length - 1),
                                    quoteType = quotes.indexOf(firstChar);
                                if (quoteType === -1) return false;
                                if (firstChar !== lastChar) return false;
                                for (var i = 0; i < string.length; i++) {
                                    if (i > 0 && i < string.length - 1)
                                        if (string.charAt(i) === quotes[quoteType])
                                            if (string.charAt(i - 1) !== '\\')
                                                return false;
                                }
                                break;
                            case 'key' :
                                if (string.length === 0) return false;
                                firstChar = string.charAt(0),
                                    lastChar = string.charAt(string.length - 1),
                                    quoteType = quotes.indexOf(firstChar);
                                if (quoteType > -1) {
                                    if (string.length === 1) return false;
                                    if (firstChar !== lastChar) return false;
                                    for (var i = 0; i < string.length; i++) {
                                        if (i > 0 && i < string.length - 1)
                                            if (string.charAt(i) === quotes[quoteType])
                                                if (string.charAt(i - 1) !== '\\')
                                                    return false;
                                    }
                                } else {
                                    const nonAlphanumeric = '\'"`.,:;{}[]&<>=~*%\\|/-+!?@^ \xa0';
                                    for (var i = 0; i < nonAlphanumeric.length; i++) {
                                        const nonAlpha = nonAlphanumeric.charAt(i);
                                        if (string.indexOf(nonAlpha) > -1) return false;
                                    }
                                }
                                break;
                            case 'number' :
                                for (var i = 0; i < string.length; i++) {
                                    if ('0123456789'.indexOf(string.charAt(i)) === -1)
                                        if (i === 0) {
                                            if ('-' !== string.charAt(0)) return false;
                                        }
                                        else if ('.' !== string.charAt(i)) return false;
                                }
                                break;
                            case 'symbol' :
                                if (string.length > 1) return false;
                                if ('{[:]},'.indexOf(string) === -1) return false;
                                break;
                            case 'colon' :
                                if (string.length > 1) return false;
                                if (':' !== string) return false;
                                break;
                            default :
                                return true;
                                break;
                        }
                        return true;
                    }

                    for (var i = 0; i < buffer.tokens_proto.length; i++) {
                        let token = buffer.tokens_proto[i];
                        if (token.type.indexOf('proto') === -1) {
                            if (!validToken(token.string, token.type)) {
                                buffer.tokens_split = buffer.tokens_split.concat(quarkize(token.string, 'split'));
                            } else buffer.tokens_split.push(token);
                        } else buffer.tokens_split.push(token);
                    }
                    for (var i = 0; i < buffer.tokens_split.length; i++) {
                        let token = buffer.tokens_split[i];
                        let
                            type = token.type,
                            string = token.string,
                            length = string.length,
                            fallback = [];
                        if (type.indexOf('-') > -1) {
                            type = type.slice(type.indexOf('-') + 1);
                            if (type !== 'string') fallback.push('string');
                            fallback.push('key');
                            fallback.push('error');
                        }
                        let tokul = {
                            string: string,
                            length: length,
                            type: type,
                            fallback: fallback
                        };
                        buffer.tokens_fallback.push(tokul);
                    }

                    function tokenFollowed() {
                        const last = buffer.tokens_normalize.length - 1;
                        if (last < 1) return false;
                        for (var i = last; i >= 0; i--) {
                            const previousToken = buffer.tokens_normalize[i];
                            switch (previousToken.type) {
                                case 'space' :
                                case 'linebreak' :
                                    break;
                                default :
                                    return previousToken;
                                    break;
                            }
                        }
                        return false;
                    }

                    let buffer2 = {
                        brackets: [],
                        stringOpen: false,
                        isValue: false
                    };
                    for (var i = 0; i < buffer.tokens_fallback.length; i++) {
                        let token = buffer.tokens_fallback[i];
                        const
                            type = token.type,
                            string = token.string;
                        let normalToken = {
                            type: type,
                            string: string
                        };
                        switch (type) {
                            case 'symbol' :
                            case 'colon' :
                                if (buffer2.stringOpen) {
                                    if (buffer2.isValue) normalToken.type = 'string';
                                    else normalToken.type = 'key';
                                    break;
                                }
                                switch (string) {
                                    case '[' :
                                    case '{' :
                                        buffer2.brackets.push(string);
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        break;
                                    case ']' :
                                    case '}' :
                                        buffer2.brackets.pop();
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        break;
                                    case ',' :
                                        if (tokenFollowed().type === 'colon') break;
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        break;
                                    case ':' :
                                        normalToken.type = 'colon';
                                        buffer2.isValue = true;
                                        break;
                                }
                                break;
                            case 'delimiter' :
                                if (buffer2.isValue) normalToken.type = 'string';
                                else normalToken.type = 'key';
                                if (!buffer2.stringOpen) {
                                    buffer2.stringOpen = string;
                                    break;
                                }
                                if (i > 0) {
                                    const
                                        previousToken = buffer.tokens_fallback[i - 1],
                                        _string = previousToken.string,
                                        _type = previousToken.type,
                                        _char = _string.charAt(_string.length - 1);
                                    if (_type === 'string' && _char === '\\') break;
                                }
                                if (buffer2.stringOpen === string) {
                                    buffer2.stringOpen = false;
                                    break;
                                }
                                break;
                            case 'primitive' :
                            case 'string' :
                                if (['false', 'true', 'null', 'undefined'].indexOf(string) > -1) {
                                    const lastIndex = buffer.tokens_normalize.length - 1;
                                    if (lastIndex >= 0) {
                                        if (buffer.tokens_normalize[lastIndex].type !== 'string') {
                                            normalToken.type = 'primitive';
                                            break;
                                        }
                                        normalToken.type = 'string';
                                        break;
                                    }
                                    normalToken.type = 'primitive';
                                    break;
                                }
                                if (string === '\n') if (!buffer2.stringOpen) {
                                    normalToken.type = 'linebreak';
                                    break;
                                }
                                if (buffer2.isValue) normalToken.type = 'string';
                                else normalToken.type = 'key';
                                break;
                            case 'space' :
                                if (buffer2.stringOpen)
                                    if (buffer2.isValue) normalToken.type = 'string';
                                    else normalToken.type = 'key';
                                break;
                            case 'number' :
                                if (buffer2.stringOpen)
                                    if (buffer2.isValue) normalToken.type = 'string';
                                    else normalToken.type = 'key';
                                break;
                            default :

                                break;
                        }
                        buffer.tokens_normalize.push(normalToken);
                    }
                    for (var i = 0; i < buffer.tokens_normalize.length; i++) {
                        const token = buffer.tokens_normalize[i];
                        let mergedToken = {
                            string: token.string,
                            type: token.type,
                            tokens: [i]
                        };
                        if (['symbol', 'colon'].indexOf(token.type) === -1)
                            if (i + 1 < buffer.tokens_normalize.length) {
                                let count = 0;
                                for (var u = i + 1; u < buffer.tokens_normalize.length; u++) {
                                    const nextToken = buffer.tokens_normalize[u];
                                    if (token.type !== nextToken.type) break;
                                    mergedToken.string += nextToken.string;
                                    mergedToken.tokens.push(u);
                                    count++;
                                }
                                i += count;
                            }
                        buffer.tokens_merge.push(mergedToken);
                    }
                    const
                        quotes = '\'"',
                        alphanumeric = (
                            'abcdefghijklmnopqrstuvwxyz' +
                            'ABCDEFGHIJKLMNOPQRSTUVWXYZ' +
                            '0123456789' +
                            '_$'
                        );
                    var
                        error = false,
                        line = buffer.tokens_merge.length > 0 ? 1 : 0;
                    buffer2 = {
                        brackets: [],
                        stringOpen: false,
                        isValue: false
                    };

                    function setError(tokenID, reason, offset = 0) {
                        console.log('>>>>>tokenID', tokenID);
                        console.log('>>>>>reason', reason);
                        error = {
                            token: tokenID,
                            line: line,
                            reason: reason
                        };
                        buffer.tokens_merge[tokenID + offset].type = 'error';
                    }

                    function followedBySymbol(tokenID, options) {
                        if (tokenID === undefined) console.error('tokenID argument must be an integer.');
                        if (options === undefined) console.error('options argument must be an array.');
                        if (tokenID === buffer.tokens_merge.length - 1) return false;
                        for (var i = tokenID + 1; i < buffer.tokens_merge.length; i++) {
                            const nextToken = buffer.tokens_merge[i];
                            switch (nextToken.type) {
                                case 'space' :
                                case 'linebreak' :
                                    break;
                                case 'symbol' :
                                case 'colon' :
                                    if (options.indexOf(nextToken.string) > -1) return i;
                                    else return false;
                                    break;
                                default :
                                    return false;
                                    break;
                            }
                        }
                        return false;
                    }

                    function followsSymbol(tokenID, options) {
                        if (tokenID === undefined) console.error('tokenID argument must be an integer.');
                        if (options === undefined) console.error('options argument must be an array.');
                        if (tokenID === 0) return false;
                        for (var i = tokenID - 1; i >= 0; i--) {
                            const previousToken = buffer.tokens_merge[i];
                            switch (previousToken.type) {
                                case 'space' :
                                case 'linebreak' :
                                    break;
                                case 'symbol' :
                                case 'colon' :
                                    if (options.indexOf(previousToken.string) > -1) return true;
                                    return false;
                                    break;
                                default :
                                    return false;
                                    break;
                            }
                        }
                        return false;
                    }

                    function typeFollowed(tokenID) {
                        if (tokenID === undefined) console.error('tokenID argument must be an integer.');
                        if (tokenID === 0) return false;
                        for (var i = tokenID - 1; i >= 0; i--) {
                            const previousToken = buffer.tokens_merge[i];
                            switch (previousToken.type) {
                                case 'space' :
                                case 'linebreak' :
                                    break;
                                default :
                                    return previousToken.type;
                                    break;
                            }
                        }
                        return false;
                    }

                    let bracketList = [];
                    for (var i = 0; i < buffer.tokens_merge.length; i++) {
                        if (error) break;
                        let
                            token = buffer.tokens_merge[i],
                            string = token.string,
                            type = token.type,
                            found = false;
                        switch (type) {
                            case 'space' :
                                break;
                            case 'linebreak' :
                                line++;
                                break;
                            case 'symbol' :
                                switch (string) {
                                    case '{' :
                                    case '[' :
                                        found = followsSymbol(i, ['}', ']']);
                                        if (found) {
                                            setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                                firstToken: buffer.tokens_merge[found].string,
                                                secondToken: string
                                            }));
                                            break;
                                        }
                                        if (string === '[' && i > 0)
                                            if (!followsSymbol(i, [':', '[', ','])) {
                                                setError(i, format(locale.invalidToken.tokenSequence.permitted, {
                                                    firstToken: '[',
                                                    secondToken: [':', '[', ',']
                                                }));
                                                break;
                                            }
                                        if (string === '{')
                                            if (followsSymbol(i, ['{'])) {
                                                setError(i, format(locale.invalidToken.double, {
                                                    token: '{'
                                                }));
                                                break;
                                            }
                                        buffer2.brackets.push(string);
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        bracketList.push({i: i, line: line, string: string});
                                        break;
                                    case '}' :
                                    case ']' :
                                        if (string === '}')
                                            if (buffer2.brackets[buffer2.brackets.length - 1] !== '{') {
                                                setError(i, format(locale.brace.curly.missingOpen));
                                                break;
                                            }
                                        if (string === '}')
                                            if (followsSymbol(i, [','])) {
                                                setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                                    firstToken: ',',
                                                    secondToken: '}'
                                                }));
                                                break;
                                            }
                                        if (string === ']')
                                            if (buffer2.brackets[buffer2.brackets.length - 1] !== '[') {
                                                setError(i, format(locale.brace.square.missingOpen));
                                                break;
                                            }
                                        if (string === ']')
                                            if (followsSymbol(i, [':'])) {
                                                setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                                    firstToken: ':',
                                                    secondToken: ']'
                                                }));
                                                break;
                                            }
                                        buffer2.brackets.pop();
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        bracketList.push({i: i, line: line, string: string});
                                        break;
                                    case ',' :
                                        found = followsSymbol(i, ['{']);
                                        if (found) {
                                            if (followedBySymbol(i, ['}'])) {
                                                setError(i, format(locale.brace.curly.cannotWrap, {
                                                    token: ','
                                                }));
                                                break;
                                            }
                                            setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                                firstToken: '{',
                                                secondToken: ','
                                            }));
                                            break;
                                        }
                                        if (followedBySymbol(i, ['}', ',', ']'])) {
                                            setError(i, format(locale.noTrailingOrLeadingComma));
                                            break;
                                        }
                                        found = typeFollowed(i);
                                        switch (found) {
                                            case 'key' :
                                            case 'colon' :
                                                setError(i, format(locale.invalidToken.termSequence.prohibited, {
                                                    firstTerm: found === 'key' ? locale.types.key : locale.symbols.colon,
                                                    secondTerm: locale.symbols.comma
                                                }));
                                                break;
                                            case 'symbol' :
                                                if (followsSymbol(i, ['{'])) {
                                                    setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                                        firstToken: '{',
                                                        secondToken: ','
                                                    }));
                                                    break;
                                                }
                                                break;
                                            default :
                                                break;
                                        }
                                        buffer2.isValue = buffer2.brackets[buffer2.brackets.length - 1] === '[';
                                        break;
                                    default :
                                        break;
                                }
                                buffer.json += string;
                                break;
                            case 'colon' :
                                found = followsSymbol(i, ['[']);
                                if (found && followedBySymbol(i, [']'])) {
                                    setError(i, format(locale.brace.square.cannotWrap, {
                                        token: ':'
                                    }));
                                    break;
                                }
                                if (found) {
                                    setError(i, format(locale.invalidToken.tokenSequence.prohibited, {
                                        firstToken: '[',
                                        secondToken: ':'
                                    }));
                                    break;
                                }
                                if (typeFollowed(i) !== 'key') {
                                    setError(i, format(locale.invalidToken.termSequence.permitted, {
                                        firstTerm: locale.symbols.colon,
                                        secondTerm: locale.types.key
                                    }));
                                    break;
                                }
                                if (followedBySymbol(i, ['}', ']'])) {
                                    setError(i, format(locale.invalidToken.termSequence.permitted, {
                                        firstTerm: locale.symbols.colon,
                                        secondTerm: locale.types.value
                                    }));
                                    break;
                                }
                                buffer2.isValue = true;
                                buffer.json += string;
                                break;
                            case 'key' :
                            case 'string' :
                                let
                                    firstChar = string.charAt(0),
                                    lastChar = string.charAt(string.length - 1),
                                    quote_primary = quotes.indexOf(firstChar);
                                if (quotes.indexOf(firstChar) === -1)
                                    if (quotes.indexOf(lastChar) !== -1) {
                                        setError(i, format(locale.string.missingOpen, {
                                            quote: firstChar
                                        }));
                                        break;
                                    }
                                if (quotes.indexOf(lastChar) === -1)
                                    if (quotes.indexOf(firstChar) !== -1) {
                                        setError(i, format(locale.string.missingClose, {
                                            quote: firstChar,
                                        }));
                                        break;
                                    }
                                if (quotes.indexOf(firstChar) > -1)
                                    if (firstChar !== lastChar) {
                                        setError(i, format(locale.string.missingClose, {
                                            quote: firstChar,
                                        }));
                                        break;
                                    }
                                if ('string' === type)
                                    if (quotes.indexOf(firstChar) === -1 && quotes.indexOf(lastChar) === -1) {
                                        setError(i, format(locale.string.mustBeWrappedByQuotes));
                                        break;
                                    }
                                if ('key' === type)
                                    if (followedBySymbol(i, ['}', ']'])) {
                                        setError(i, format(locale.invalidToken.termSequence.permitted, {
                                            firstTerm: locale.types.key,
                                            secondTerm: locale.symbols.colon
                                        }));
                                    }
                                if (quotes.indexOf(firstChar) === -1 && quotes.indexOf(lastChar) === -1)
                                    for (var h = 0; h < string.length; h++) {
                                        if (error) break;
                                        const c = string.charAt(h);
                                        if (alphanumeric.indexOf(c) === -1) {
                                            setError(i, format(locale.string.nonAlphanumeric, {
                                                token: c,
                                            }));
                                            break;
                                        }
                                    }
                                if (firstChar === '\'') string = '"' + string.slice(1, -1) + '"';
                                else if (firstChar !== '"') string = '"' + string + '"';
                                if ('key' === type)
                                    if ('key' === typeFollowed(i)) {
                                        if (i > 0)
                                            if (!isNaN(buffer.tokens_merge[i - 1])) {
                                                buffer.tokens_merge[i - 1] += buffer.tokens_merge[i];
                                                setError(i, format(locale.key.numberAndLetterMissingQuotes));
                                                break;
                                            }
                                        setError(i, format(locale.key.spaceMissingQuotes));
                                        break;
                                    }
                                if ('key' === type)
                                    if (!followsSymbol(i, ['{', ','])) {
                                        setError(i, format(locale.invalidToken.tokenSequence.permitted, {
                                            firstToken: type,
                                            secondToken: ['{', ',']
                                        }));
                                        break;
                                    }
                                if ('string' === type)
                                    if (!followsSymbol(i, ['[', ':', ','])) {
                                        setError(i, format(locale.invalidToken.tokenSequence.permitted, {
                                            firstToken: type,
                                            secondToken: ['[', ':', ',']
                                        }));
                                        break;
                                    }
                                if ('key' === type)
                                    if (buffer2.isValue) {
                                        setError(i, format(locale.string.unexpectedKey));
                                        break;
                                    }
                                if ('string' === type)
                                    if (!buffer2.isValue) {
                                        setError(i, format(locale.key.unexpectedString));
                                        break;
                                    }
                                buffer.json += string;
                                break;
                            case 'number' :
                            case 'primitive' :
                                if (followsSymbol(i, ['{'])) {
                                    buffer.tokens_merge[i].type = 'key';
                                    type = buffer.tokens_merge[i].type;
                                    string = '"' + string + '"';
                                }
                                else if (typeFollowed(i) === 'key') {
                                    buffer.tokens_merge[i].type = 'key';
                                    type = buffer.tokens_merge[i].type;
                                }
                                else if (!followsSymbol(i, ['[', ':', ','])) {
                                    setError(i, format(locale.invalidToken.tokenSequence.permitted, {
                                        firstToken: type,
                                        secondToken: ['[', ':', ',']
                                    }));
                                    break;
                                }
                                if (type !== 'key')
                                    if (!buffer2.isValue) {
                                        buffer.tokens_merge[i].type = 'key';
                                        type = buffer.tokens_merge[i].type;
                                        string = '"' + string + '"';
                                    }
                                if (type === 'primitive')
                                    if (string === 'undefined')
                                        setError(i, format(locale.invalidToken.useInstead, {
                                            badToken: 'undefined',
                                            goodToken: 'null'
                                        }));
                                buffer.json += string;
                                break;
                        }
                    }
                    let noEscapedSingleQuote = '';
                    for (var i = 0; i < buffer.json.length; i++) {
                        let
                            current = buffer.json.charAt(i),
                            next = '';
                        if (i + 1 < buffer.json.length) {
                            next = buffer.json.charAt(i + 1);
                            if (current === '\\' && next === '\'') {
                                noEscapedSingleQuote += next;
                                i++;
                                continue;
                            }
                        }
                        noEscapedSingleQuote += current;
                    }
                    buffer.json = noEscapedSingleQuote;
                    if (!error) {
                        const maxIterations = Math.ceil(bracketList.length / 2);
                        let
                            round = 0,
                            delta = false;

                        function removePair(index) {
                            bracketList.splice(index + 1, 1);
                            bracketList.splice(index, 1);
                            if (!delta) delta = true;
                        }

                        while (bracketList.length > 0) {
                            delta = false;
                            for (var tokenCount = 0; tokenCount < bracketList.length - 1; tokenCount++) {
                                const pair = bracketList[tokenCount].string + bracketList[tokenCount + 1].string;
                                if (['[]', '{}'].indexOf(pair) > -1) removePair(tokenCount);
                            }
                            round++;
                            if (!delta) break;
                            if (round >= maxIterations) break;
                        }
                        if (bracketList.length > 0) {
                            const
                                _tokenString = bracketList[0].string,
                                _tokenPosition = bracketList[0].i,
                                _closingBracketType = _tokenString === '[' ? ']' : '}';
                            line = bracketList[0].line;
                            setError(_tokenPosition, format(locale.brace[_closingBracketType === ']' ? 'square' : 'curly'].missingClose));
                        }
                    }
                    if (!error)
                        if ([undefined, ''].indexOf(buffer.json) === -1)
                            try {
                                buffer.jsObject = JSON.parse(buffer.json);
                            }
                            catch (err) {
                                const
                                    errorMessage = err.message,
                                    subsMark = errorMessage.indexOf('position');
                                if (subsMark === -1) throw new Error('Error parsing failed');
                                const
                                    errPositionStr = errorMessage.substring(subsMark + 9, errorMessage.length),
                                    errPosition = parseInt(errPositionStr);
                                let
                                    charTotal = 0,
                                    tokenIndex = 0,
                                    token = false,
                                    _line = 1,
                                    exitWhile = false;
                                while (charTotal < errPosition && !exitWhile) {
                                    token = buffer.tokens_merge[tokenIndex];
                                    if ('linebreak' === token.type) _line++;
                                    if (['space', 'linebreak'].indexOf(token.type) === -1) charTotal += token.string.length;
                                    if (charTotal >= errPosition) break;
                                    tokenIndex++;
                                    if (!buffer.tokens_merge[tokenIndex + 1]) exitWhile = true;
                                }
                                line = _line;
                                let backslashCount = 0;
                                for (let i = 0; i < token.string.length; i++) {
                                    const char = token.string.charAt(i);
                                    if (char === '\\')
                                        backslashCount = backslashCount > 0 ? backslashCount + 1 : 1;
                                    else {
                                        if (backslashCount % 2 !== 0 || backslashCount === 0)
                                            if ('\'"bfnrt'.indexOf(char) === -1) {
                                                setError(tokenIndex, format(locale.invalidToken.unexpected, {
                                                    token: '\\'
                                                }));
                                            }
                                        backslashCount = 0;
                                    }
                                }
                                if (!error)
                                    setError(tokenIndex, format(locale.invalidToken.unexpected, {
                                        token: token.string
                                    }));
                            }
                    let
                        _line = 1,
                        _depth = 0;

                    function newIndent() {
                        var space = [];
                        for (var i = 0; i < _depth * 2; i++) space.push('&nbsp;');
                        return space.join('');
                    }

                    function newLineBreak(byPass = false) {
                        _line++;
                        if (_depth > 0 || byPass) {
                            return '<br>';
                        }
                        return '';
                    }

                    function newLineBreakAndIndent(byPass = false) {
                        return newLineBreak(byPass) + newIndent();
                    };
                    if (!error)
                        for (var i = 0; i < buffer.tokens_merge.length; i++) {
                            const
                                token = buffer.tokens_merge[i],
                                string = token.string,
                                type = token.type;
                            switch (type) {
                                case 'space' :
                                case 'linebreak' :
                                    break;
                                case 'string' :
                                case 'number'    :
                                case 'primitive' :
                                case 'error' :
                                    buffer.markup += ((followsSymbol(i, [',', '[']) ? newLineBreakAndIndent() : '') + newSpan(i, token, _depth));
                                    break;
                                case 'key' :
                                    buffer.markup += (newLineBreakAndIndent() + newSpan(i, token, _depth));
                                    break;
                                case 'colon' :
                                    buffer.markup += (newSpan(i, token, _depth) + '&nbsp;');
                                    break;
                                case 'symbol' :
                                    switch (string) {
                                        case '[' :
                                        case '{' :
                                            buffer.markup += ((!followsSymbol(i, [':']) ? newLineBreakAndIndent() : '') + newSpan(i, token, _depth));
                                            _depth++;
                                            break;
                                        case ']' :
                                        case '}' :
                                            _depth--;
                                            const
                                                islastToken = i === buffer.tokens_merge.length - 1,
                                                _adjustment = i > 0 ? ['[', '{'].indexOf(buffer.tokens_merge[i - 1].string) > -1 ? '' : newLineBreakAndIndent(islastToken) : '';
                                            buffer.markup += (_adjustment + newSpan(i, token, _depth));
                                            break;
                                        case ',' :
                                            buffer.markup += newSpan(i, token, _depth);
                                            break;
                                    }
                                    break;
                            }
                        }
                    if (error) {
                        let _line_fallback = 1;

                        function countCarrigeReturn(string) {
                            let count = 0;
                            for (var i = 0; i < string.length; i++) {
                                if (['\n', '\r'].indexOf(string[i]) > -1) count++;
                            }
                            return count;
                        }

                        _line = 1;
                        for (var i = 0; i < buffer.tokens_merge.length; i++) {
                            const
                                token = buffer.tokens_merge[i],
                                type = token.type,
                                string = token.string;
                            if (type === 'linebreak') _line++;
                            buffer.markup += newSpan(i, token, _depth);
                            _line_fallback += countCarrigeReturn(string);
                        }
                        _line++;
                        _line_fallback++;
                        if (_line < _line_fallback) _line = _line_fallback;
                    }
                    for (var i = 0; i < buffer.tokens_merge.length; i++) {
                        let token = buffer.tokens_merge[i];
                        buffer.indented += token.string;
                        if (['space', 'linebreak'].indexOf(token.type) === -1) buffer.tokens_plainText += token.string;
                    }
                    if (error) {
                        function isFunction(functionToCheck) {
                            return functionToCheck && {}.toString.call(functionToCheck) === '[object Function]';
                        }

                        // if('modifyErrorText' in this.props)
                        //     if(isFunction(this.props.modifyErrorText))
                        //         error.reason = this.props.modifyErrorText(error.reason);
                    }
                    return {
                        tokens: buffer.tokens_merge,
                        noSpaces: buffer.tokens_plainText,
                        indented: buffer.indented,
                        json: buffer.json,
                        jsObject: buffer.jsObject,
                        markup: buffer.markup,
                        lines: _line,
                        error: error
                    };
                }
                ;

                /**
                 *     JS OBJECTS || PLACEHOLDER
                 */

                if (!('nodeType' in something)) {
                    let buffer = {
                        inputText: JSON.stringify(something),
                        position: 0,
                        currentChar: '',
                        tokenPrimary: '',
                        tokenSecondary: '',
                        brackets: [],
                        isValue: false,
                        stringOpen: false,
                        stringStart: 0,
                        tokens: []
                    };

                    function escape_character() {
                        if (buffer.currentChar !== '\\') return false;
                        buffer.inputText = deleteCharAt(buffer.inputText, buffer.position);
                        return true;
                    }

                    function deleteCharAt(string, position) {
                        return string.slice(0, position) + string.slice(position + 1);
                    }

                    function determine_string() {
                        if ('\'"'.indexOf(buffer.currentChar) === -1) return false;
                        if (!buffer.stringOpen) {
                            add_tokenSecondary();
                            buffer.stringStart = buffer.position;
                            buffer.stringOpen = buffer.currentChar;
                            return true;
                        }
                        if (buffer.stringOpen === buffer.currentChar) {
                            add_tokenSecondary();
                            const stringToken = buffer.inputText.substring(buffer.stringStart, buffer.position + 1);
                            add_tokenPrimary(stringToken);
                            buffer.stringOpen = false;
                            return true;
                        }
                        return false;
                    }

                    function determine_value() {
                        if (':,{}[]'.indexOf(buffer.currentChar) === -1) return false;
                        if (buffer.stringOpen) return false;
                        add_tokenSecondary();
                        add_tokenPrimary(buffer.currentChar);
                        switch (buffer.currentChar) {
                            case ':' :
                                buffer.isValue = true;
                                return true;
                                break;
                            case '{' :
                            case '[' :
                                buffer.brackets.push(buffer.currentChar);
                                break;
                            case '}' :
                            case ']' :
                                buffer.brackets.pop();
                                break;
                        }
                        if (buffer.currentChar !== ':') buffer.isValue = (buffer.brackets[buffer.brackets.length - 1] === '[');
                        return true;
                    }

                    function add_tokenSecondary() {
                        if (buffer.tokenSecondary.length === 0) return false;
                        buffer.tokens.push(buffer.tokenSecondary);
                        buffer.tokenSecondary = '';
                        return true;
                    }

                    function add_tokenPrimary(value) {
                        if (value.length === 0) return false;
                        buffer.tokens.push(value);
                        return true;
                    }

                    for (var i = 0; i < buffer.inputText.length; i++) {
                        buffer.position = i;
                        buffer.currentChar = buffer.inputText.charAt(buffer.position);
                        const
                            a = determine_value(),
                            b = determine_string(),
                            c = escape_character();
                        if (!a && !b && !c) if (!buffer.stringOpen) buffer.tokenSecondary += buffer.currentChar;
                    }
                    let buffer2 = {brackets: [], isValue: false, tokens: []};
                    buffer2.tokens = buffer.tokens.map(token => {
                        let
                            type = '',
                            string = '',
                            value = '';
                        switch (token) {
                            case ',' :
                                type = 'symbol';
                                string = token;
                                value = token;
                                buffer2.isValue = (buffer2.brackets[buffer2.brackets.length - 1] === '[');
                                break;
                            case ':' :
                                type = 'symbol';
                                string = token;
                                value = token;
                                buffer2.isValue = true;
                                break;
                            case '{' :
                            case '[' :
                                type = 'symbol';
                                string = token;
                                value = token;
                                buffer2.brackets.push(token);
                                buffer2.isValue = (buffer2.brackets[buffer2.brackets.length - 1] === '[');
                                break;
                            case '}' :
                            case ']' :
                                type = 'symbol';
                                string = token;
                                value = token;
                                buffer2.brackets.pop();
                                buffer2.isValue = (buffer2.brackets[buffer2.brackets.length - 1] === '[');
                                break;
                            case 'undefined' :
                                type = 'primitive';
                                string = token;
                                value = undefined;
                                break;
                            case 'null' :
                                type = 'primitive';
                                string = token;
                                value = null;
                                break;
                            case 'false' :
                                type = 'primitive';
                                string = token;
                                value = false;
                                break;
                            case 'true' :
                                type = 'primitive';
                                string = token;
                                value = true;
                                break;
                            default :
                                const C = token.charAt(0);

                            function stripQuotesFromKey(text) {
                                if (text.length === 0) return text;
                                if (['""', '\'\''].indexOf(text) > -1) return '\'\'';
                                let wrappedInQuotes = false;
                                for (var i = 0; i < 2; i++) {
                                    if ([text.charAt(0), text.charAt(text.length - 1)].indexOf(['"', '\''][i]) > -1) {
                                        wrappedInQuotes = true;
                                        break;
                                    }
                                }
                                if (wrappedInQuotes && text.length >= 2) text = text.slice(1, -1);
                                const
                                    nonAlphaNumeric = text.replace(/\w/g, ''),
                                    alphaNumeric = text.replace(/\W+/g, ''),
                                    mayRemoveQuotes = ((nonAlphaNumeric, text) => {
                                        let numberAndLetter = false;
                                        for (var i = 0; i < text.length; i++) {
                                            if (i === 0) if (isNaN(text.charAt(i))) break;
                                            if (isNaN(text.charAt(i))) {
                                                numberAndLetter = true;
                                                break;
                                            }
                                        }
                                        return !(nonAlphaNumeric.length > 0 || numberAndLetter);
                                    })(nonAlphaNumeric, text),
                                    hasQuotes = (string => {
                                        for (var i = 0; i < string.length; i++) {
                                            if (['\'', '"'].indexOf(string.charAt(i)) > -1) return true;
                                        }
                                        return false;
                                    })(nonAlphaNumeric);
                                if (hasQuotes) {
                                    let newText = '';
                                    const charList = text.split('');
                                    for (var ii = 0; ii < charList.length; ii++) {
                                        let char = charList[ii];
                                        if (['\'', '"'].indexOf(char) > -1) char = '\\' + char;
                                        newText += char;
                                    }
                                    text = newText;
                                }
                                if (!mayRemoveQuotes)
                                    return '\'' + text + '\'';
                                else
                                    return text;
                            }

                                if ('\'"'.indexOf(C) > -1) {
                                    if (buffer2.isValue) type = 'string'; else type = 'key';
                                    if (type === 'key') string = stripQuotesFromKey(token);
                                    if (type === 'string') {
                                        string = '';
                                        const charList2 = token.slice(1, -1).split('');
                                        for (var ii = 0; ii < charList2.length; ii++) {
                                            let char = charList2[ii];
                                            if ('\'\"'.indexOf(char) > -1) char = '\\' + char;
                                            string += char;
                                        }
                                        string = '\'' + string + '\'';
                                    }
                                    value = string;
                                    break;
                                }
                                if (!isNaN(token)) {
                                    type = 'number';
                                    string = token;
                                    value = Number(token);
                                    break;
                                }
                                if (token.length > 0)
                                    if (!buffer2.isValue) {
                                        type = 'key';
                                        string = token;
                                        if (string.indexOf(' ') > -1) string = '\'' + string + '\'';
                                        value = string;
                                        break;
                                    }
                        }
                        return {
                            type: type,
                            string: string,
                            value: value,
                            depth: buffer2.brackets.length
                        };
                    });
                    let clean = '';
                    for (var i = 0; i < buffer2.tokens.length; i++) {
                        let token = buffer2.tokens[i];
                        clean += token.string;
                    }

                    function indent(number) {
                        var space = [];
                        for (var i = 0; i < number * 2; i++) space.push(' ');
                        return (number > 0 ? '\n' : '') + space.join('');
                    };
                    let indentation = '';
                    for (var i = 0; i < buffer2.tokens.length; i++) {
                        let token = buffer2.tokens[i];
                        switch (token.string) {
                            case '[' :
                            case '{' :
                                const nextToken = i < (buffer2.tokens.length - 1) - 1 ? buffer2.tokens[i + 1] : '';
                                if ('}]'.indexOf(nextToken.string) === -1)
                                    indentation += token.string + indent(token.depth);
                                else
                                    indentation += token.string;
                                break;
                            case ']' :
                            case '}' :
                                const prevToken = i > 0 ? buffer2.tokens[i - 1] : '';
                                if ('[{'.indexOf(prevToken.string) === -1)
                                    indentation += indent(token.depth) + token.string;
                                else
                                    indentation += token.string;
                                break;
                            case ':' :
                                indentation += token.string + ' ';
                                break;
                            case ',' :
                                indentation += token.string + indent(token.depth);
                                break;
                            default :
                                indentation += token.string;
                                break;
                        }
                    }
                    let lines = 1;

                    function indentII(number) {
                        var space = [];
                        if (number > 0) lines++;
                        for (var i = 0; i < number * 2; i++) space.push('&nbsp;');
                        return (number > 0 ? '<br>' : '') + space.join('');
                    };
                    let markup = '';
                    const lastIndex = buffer2.tokens.length - 1;
                    for (var i = 0; i < buffer2.tokens.length; i++) {
                        let token = buffer2.tokens[i];
                        let span = newSpan(i, token, token.depth);
                        switch (token.string) {
                            case '{' :
                            case '[' :
                                const nextToken = i < (buffer2.tokens.length - 1) - 1 ? buffer2.tokens[i + 1] : '';
                                if ('}]'.indexOf(nextToken.string) === -1)
                                    markup += span + indentII(token.depth);
                                else
                                    markup += span;
                                break;
                            case '}' :
                            case ']' :
                                const prevToken = i > 0 ? buffer2.tokens[i - 1] : '';
                                if ('[{'.indexOf(prevToken.string) === -1)
                                    markup += indentII(token.depth) + (lastIndex === i ? '<br>' : '') + span;
                                else
                                    markup += span;
                                break;
                            case ':' :
                                markup += span + ' ';
                                break;
                            case ',' :
                                markup += span + indentII(token.depth);
                                break;
                            default  :
                                markup += span;
                                break;
                        }
                    }
                    lines += 2;
                    return {
                        tokens: buffer2.tokens,
                        noSpaces: clean,
                        indented: indentation,
                        json: JSON.stringify(something),
                        jsObject: something,
                        markup: markup,
                        lines: lines
                    };
                }
            }
        },
        mounted() {

            this.updateTime = false;
            this.timer = setInterval(this.scheduledUpdate, 100);
            this.initData(this.value)
        },
        beforeDestroy() {
            if (this.timer) clearInterval(this.timer)
        }
    };
</script>

<style scoped>
    @import "styles.css";
</style>
