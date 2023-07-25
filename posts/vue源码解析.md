---
title: Vue源码解析
description: vue源码解析
date: 2022-5-20
tags:
  - Vue
---
## 1~209

```js
/*!

 * Vue.js v2.7.14

 * (c) 2014-2022 Evan You

 * Released under the MIT License.

 */

var emptyObject = Object.freeze({});

var isArray = Array.isArray;

// These helpers produce better VM code in JS engines due to their

// explicitness and function inlining.

function isUndef(v) {
  return v === undefined || v === null;
}

function isDef(v) {
  return v !== undefined && v !== null;
}

function isTrue(v) {
  return v === true;
}

function isFalse(v) {
  return v === false;
}

/**

 * Check if value is primitive.

 */

function isPrimitive(value) {
  return (
    typeof value === "string" ||
    typeof value === "number" || // $flow-disable-line
    typeof value === "symbol" ||
    typeof value === "boolean"
  );
}

function isFunction(value) {
  return typeof value === "function";
}

/**

 * Quick object check - this is primarily used to tell

 * objects from primitive values when we know the value

 * is a JSON-compliant type.

 */

function isObject(obj) {
  return obj !== null && typeof obj === "object";
}

/**

 * Get the raw type string of a value, e.g., [object Object].

 */

var _toString = Object.prototype.toString;

function toRawType(value) {
  return _toString.call(value).slice(8, -1);
}

/**

 * Strict object type check. Only returns true

 * for plain JavaScript objects.

 */

function isPlainObject(obj) {
  return _toString.call(obj) === "[object Object]";
}

function isRegExp(v) {
  return _toString.call(v) === "[object RegExp]";
}

/**

 * Check if val is a valid array index.

 */

function isValidArrayIndex(val) {
  var n = parseFloat(String(val));

  return n >= 0 && Math.floor(n) === n && isFinite(val);
}

function isPromise(val) {
  return (
    isDef(val) &&
    typeof val.then === "function" &&
    typeof val.catch === "function"
  );
}

/**

 * Convert a value to a string that is actually rendered.

 */

function toString(val) {
  return val == null
    ? ""
    : Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
    ? JSON.stringify(val, null, 2)
    : String(val);
}

/**

 * Convert an input value to a number for persistence.

 * If the conversion fails, return original string.

 */

function toNumber(val) {
  var n = parseFloat(val);

  return isNaN(n) ? val : n;
}

/**

 * Make a map and return a function for checking if a key

 * is in that map.

 */

function makeMap(str, expectsLowerCase) {
  var map = Object.create(null);

  var list = str.split(",");

  for (var i = 0; i < list.length; i++) {
    map[list[i]] = true;
  }

  return expectsLowerCase
    ? function (val) {
        return map[val.toLowerCase()];
      }
    : function (val) {
        return map[val];
      };
}

/**

 * Check if a tag is a built-in tag.

 */

var isBuiltInTag = makeMap("slot,component", true);

/**

 * Check if an attribute is a reserved attribute.

 */

var isReservedAttribute = makeMap("key,ref,slot,slot-scope,is");

/**

 * Remove an item from an array.

 */

function remove$2(arr, item) {
  var len = arr.length;

  if (len) {
    // fast path for the only / last item

    if (item === arr[len - 1]) {
      arr.length = len - 1;

      return;
    }

    var index = arr.indexOf(item);

    if (index > -1) {
      return arr.splice(index, 1);
    }
  }
}

/**

 * Check whether an object has the property.

 */

var hasOwnProperty = Object.prototype.hasOwnProperty;

function hasOwn(obj, key) {
  return hasOwnProperty.call(obj, key);
}

/**

 * Create a cached version of a pure function.

 */

function cached(fn) {
  var cache = Object.create(null);

  return function cachedFn(str) {
    var hit = cache[str];

    return hit || (cache[str] = fn(str));
  };
}

/**

 * Camelize a hyphen-delimited string.

 */

var camelizeRE = /-(\w)/g;

var camelize = cached(function (str) {
  return str.replace(camelizeRE, function (_, c) {
    return c ? c.toUpperCase() : "";
  });
});

/**

 * Capitalize a string.

 */

var capitalize = cached(function (str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
});

/**

 * Hyphenate a camelCase string.

 */

var hyphenateRE = /\B([A-Z])/g;

var hyphenate = cached(function (str) {
  return str.replace(hyphenateRE, "-$1").toLowerCase();
});

/**

 * Simple bind polyfill for environments that do not support it,

 * e.g., PhantomJS 1.x. Technically, we don't need this anymore

 * since native bind is now performant enough in most browsers.

 * But removing it would mean breaking code that was able to run in

 * PhantomJS 1.x, so this must be kept for backward compatibility.

 */

/* istanbul ignore next */

function polyfillBind(fn, ctx) {
  function boundFn(a) {
    var l = arguments.length;

    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx);
  }

  boundFn._length = fn.length;

  return boundFn;
}

function nativeBind(fn, ctx) {
  return fn.bind(ctx);
}

// @ts-expect-error bind cannot be `undefined`

var bind$1 = Function.prototype.bind ? nativeBind : polyfillBind;

/**

 * Convert an Array-like object to a real Array.

 */

function toArray(list, start) {
  start = start || 0;

  var i = list.length - start;

  var ret = new Array(i);

  while (i--) {
    ret[i] = list[i + start];
  }

  return ret;
}

/**

 * Mix properties into target object.

 */

function extend(to, _from) {
  for (var key in _from) {
    to[key] = _from[key];
  }

  return to;
}
```

## 1-200 解释

这段代码是一个实用函数和帮助程序，用于处理 JavaScript 对象和数组。它包括一些用于检查值类型的函数，将值转换为字符串或数字的函数，创建映射的函数，从数组中删除项的函数以及使用另一个对象的属性扩展对象的函数。代码还包括一些字符串操作函数，用于将连字符或驼峰式字符串转换为不同格式。

# 210~400

```js
/**

 * Merge an Array of Objects into a single Object.

 */

function toObject(arr) {

    var res = {};

    for (var i = 0; i < arr.length; i++) {

        if (arr[i]) {

            extend(res, arr[i]);

        }

    }

    return res;

}

/* eslint-disable no-unused-vars */

/**

 * Perform no operation.

 * Stubbing args to make Flow happy without leaving useless transpiled code

 * with ...rest (https://flow.org/blog/2017/05/07/Strict-Function-Call-Arity/).

 */

function noop(a, b, c) { }

/**

 * Always return false.

 */

var no = function (a, b, c) { return false; };

/* eslint-enable no-unused-vars */

/**

 * Return the same value.

 */

var identity = function (_) { return _; };

/**

 * Generate a string containing static keys from compiler modules.

 */

function genStaticKeys$1(modules) {

    return modules

        .reduce(function (keys, m) {

        return keys.concat(m.staticKeys || []);

    }, [])

        .join(',');

}

/**

 * Check if two values are loosely equal - that is,

 * if they are plain objects, do they have the same shape?

 */

function looseEqual(a, b) {

    if (a === b)

        return true;

    var isObjectA = isObject(a);

    var isObjectB = isObject(b);

    if (isObjectA && isObjectB) {

        try {

            var isArrayA = Array.isArray(a);

            var isArrayB = Array.isArray(b);

            if (isArrayA && isArrayB) {

                return (a.length === b.length &&

                    a.every(function (e, i) {

                        return looseEqual(e, b[i]);

                    }));

            }

            else if (a instanceof Date && b instanceof Date) {

                return a.getTime() === b.getTime();

            }

            else if (!isArrayA && !isArrayB) {

                var keysA = Object.keys(a);

                var keysB = Object.keys(b);

                return (keysA.length === keysB.length &&

                    keysA.every(function (key) {

                        return looseEqual(a[key], b[key]);

                    }));

            }

            else {

                /* istanbul ignore next */

                return false;

            }

        }

        catch (e) {

            /* istanbul ignore next */

            return false;

        }

    }

    else if (!isObjectA && !isObjectB) {

        return String(a) === String(b);

    }

    else {

        return false;

    }

}

/**

 * Return the first index at which a loosely equal value can be

 * found in the array (if value is a plain object, the array must

 * contain an object of the same shape), or -1 if it is not present.

 */

function looseIndexOf(arr, val) {

    for (var i = 0; i < arr.length; i++) {

        if (looseEqual(arr[i], val))

            return i;

    }

    return -1;

}

/**

 * Ensure a function is called only once.

 */

function once(fn) {

    var called = false;

    return function () {

        if (!called) {

            called = true;

            fn.apply(this, arguments);

        }

    };

}

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#polyfill

function hasChanged(x, y) {

    if (x === y) {

        return x === 0 && 1 / x !== 1 / y;

    }

    else {

        return x === x || y === y;

    }

}



var SSR_ATTR = 'data-server-rendered';

var ASSET_TYPES = ['component', 'directive', 'filter'];

var LIFECYCLE_HOOKS = [

    'beforeCreate',

    'created',

    'beforeMount',

    'mounted',

    'beforeUpdate',

    'updated',

    'beforeDestroy',

    'destroyed',

    'activated',

    'deactivated',

    'errorCaptured',

    'serverPrefetch',

    'renderTracked',

    'renderTriggered'

];



var config = {

    /**

     * Option merge strategies (used in core/util/options)

     */

    // $flow-disable-line

    optionMergeStrategies: Object.create(null),

    /**

     * Whether to suppress warnings.

     */

    silent: false,

    /**

     * Show production mode tip message on boot?

     */

    productionTip: process.env.NODE_ENV !== 'production',

    /**

     * Whether to enable devtools

     */

    devtools: process.env.NODE_ENV !== 'production',

    /**

     * Whether to record perf

     */

    performance: false,

    /**

     * Error handler for watcher errors

     */

    errorHandler: null,

    /**

     * Warn handler for watcher warns

     */

    warnHandler: null,

    /**

     * Ignore certain custom elements

     */

    ignoredElements: [],

    /**

     * Custom user key aliases for v-on

     */

    // $flow-disable-line

    keyCodes: Object.create(null),

    /**

     * Check if a tag is reserved so that it cannot be registered as a

     * component. This is platform-dependent and may be overwritten.

     */

    isReservedTag: no,

    /**

     * Check if an attribute is reserved so that it cannot be used as a component

     * prop. This is platform-dependent and may be overwritten.

     */

    isReservedAttr: no,

    /**

     * Check if a tag is an unknown element.

     * Platform-dependent.

     */

    isUnknownElement: no,
```

## 201-401 解释

这是一组在 Vue.js 中使用的实用函数和配置。`toObject`函数将一个对象数组合并为一个单一的对象，而`noop`是一个什么也不做的空函数。`no`始终返回 false，而`identity`返回传递给它的相同值作为参数。`genStaticKeys$1`从编译器模块生成一个包含静态键的字符串。`looseEqual`检查两个值是否松散相等 - 即它们是否具有相同的形状的普通对象。`looseIndexOf`返回数组中第一个松散相等值的索引（如果该值是普通对象，则数组必须包含具有相同形状的对象），否则返回-1。`once`确保一个函数只被调用一次。`hasChanged`确定两个值是否已更改。最后，`config`包含由 Vue.js 使用的各种选项，例如选项合并策略、是否抑制警告、是否启用开发工具等。

# 401~600

```js
  /**

     * Get the namespace of an element

     */

    getTagNamespace: noop,

    /**

     * Parse the real tag name for the specific platform.

     */

    parsePlatformTagName: identity,

    /**

     * Check if an attribute must be bound using property, e.g. value

     * Platform-dependent.

     */

    mustUseProp: no,

    /**

     * Perform updates asynchronously. Intended to be used by Vue Test Utils

     * This will significantly reduce performance if set to false.

     */

    async: true,

    /**

     * Exposed for legacy reasons

     */

    _lifecycleHooks: LIFECYCLE_HOOKS

};



/**

 * unicode letters used for parsing html tags, component names and property paths.

 * using https://www.w3.org/TR/html53/semantics-scripting.html#potentialcustomelementname

 * skipping \u10000-\uEFFFF due to it freezing up PhantomJS

 */

var unicodeRegExp = /a-zA-Z\u00B7\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u037D\u037F-\u1FFF\u200C-\u200D\u203F-\u2040\u2070-\u218F\u2C00-\u2FEF\u3001-\uD7FF\uF900-\uFDCF\uFDF0-\uFFFD/;

/**

 * Check if a string starts with $ or _

 */

function isReserved(str) {

    var c = (str + '').charCodeAt(0);

    return c === 0x24 || c === 0x5f;

}

/**

 * Define a property.

 */

function def(obj, key, val, enumerable) {

    Object.defineProperty(obj, key, {

        value: val,

        enumerable: !!enumerable,

        writable: true,

        configurable: true

    });

}

/**

 * Parse simple path.

 */

var bailRE = new RegExp("[^".concat(unicodeRegExp.source, ".$_\\d]"));

function parsePath(path) {

    if (bailRE.test(path)) {

        return;

    }

    var segments = path.split('.');

    return function (obj) {

        for (var i = 0; i < segments.length; i++) {

            if (!obj)

                return;

            obj = obj[segments[i]];

        }

        return obj;

    };

}



// can we use __proto__?

var hasProto = '__proto__' in {};

// Browser environment sniffing

var inBrowser = typeof window !== 'undefined';

var UA = inBrowser && window.navigator.userAgent.toLowerCase();

var isIE = UA && /msie|trident/.test(UA);

var isIE9 = UA && UA.indexOf('msie 9.0') > 0;

var isEdge = UA && UA.indexOf('edge/') > 0;

UA && UA.indexOf('android') > 0;

var isIOS = UA && /iphone|ipad|ipod|ios/.test(UA);

UA && /chrome\/\d+/.test(UA) && !isEdge;

UA && /phantomjs/.test(UA);

var isFF = UA && UA.match(/firefox\/(\d+)/);

// Firefox has a "watch" function on Object.prototype...

// @ts-expect-error firebox support

var nativeWatch = {}.watch;

var supportsPassive = false;

if (inBrowser) {

    try {

        var opts = {};

        Object.defineProperty(opts, 'passive', {

            get: function () {

                /* istanbul ignore next */

                supportsPassive = true;

            }

        }); // https://github.com/facebook/flow/issues/285

        window.addEventListener('test-passive', null, opts);

    }

    catch (e) { }

}

// this needs to be lazy-evaled because vue may be required before

// vue-server-renderer can set VUE_ENV

var _isServer;

var isServerRendering = function () {

    if (_isServer === undefined) {

        /* istanbul ignore if */

        if (!inBrowser && typeof global !== 'undefined') {

            // detect presence of vue-server-renderer and avoid

            // Webpack shimming the process

            _isServer =

                global['process'] && global['process'].env.VUE_ENV === 'server';

        }

        else {

            _isServer = false;

        }

    }

    return _isServer;

};

// detect devtools

var devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__;

/* istanbul ignore next */

function isNative(Ctor) {

    return typeof Ctor === 'function' && /native code/.test(Ctor.toString());

}

var hasSymbol = typeof Symbol !== 'undefined' &&

    isNative(Symbol) &&

    typeof Reflect !== 'undefined' &&

    isNative(Reflect.ownKeys);

var _Set; // $flow-disable-line

/* istanbul ignore if */ if (typeof Set !== 'undefined' && isNative(Set)) {

    // use native Set when available.

    _Set = Set;

}

else {

    // a non-standard Set polyfill that only works with primitive keys.

    _Set = /** @class */ (function () {

        function Set() {

            this.set = Object.create(null);

        }

        Set.prototype.has = function (key) {

            return this.set[key] === true;

        };

        Set.prototype.add = function (key) {

            this.set[key] = true;

        };

        Set.prototype.clear = function () {

            this.set = Object.create(null);

        };

        return Set;

    }());

}



var currentInstance = null;

/**

 * This is exposed for compatibility with v3 (e.g. some functions in VueUse

 * relies on it). Do not use this internally, just use `currentInstance`.

 *

 * @internal this function needs manual type declaration because it relies

 * on previously manually authored types from Vue 2

 */

function getCurrentInstance() {

    return currentInstance && { proxy: currentInstance };

}

/**

 * @internal

 */

function setCurrentInstance(vm) {

    if (vm === void 0) { vm = null; }

    if (!vm)

        currentInstance && currentInstance._scope.off();

    currentInstance = vm;

    vm && vm._scope.on();

}



/**

 * @internal

 */

var VNode = /** @class */ (function () {

    function VNode(tag, data, children, text, elm, context, componentOptions, asyncFactory) {

        this.tag = tag;

        this.data = data;

        this.children = children;

        this.text = text;

        this.elm = elm;

        this.ns = undefined;

        this.context = context;

        this.fnContext = undefined;

        this.fnOptions = undefined;

        this.fnScopeId = undefined;

        this.key = data && data.key;

        this.componentOptions = componentOptions;

        this.componentInstance = undefined;

        this.parent = undefined;

        this.raw = false;

        this.isStatic = false;

        this.isRootInsert = true;

        this.isComment = false;

        this.isCloned = false;

        this.isOnce = false;

        this.asyncFactory = asyncFactory;

        this.asyncMeta = undefined;

        this.isAsyncPlaceholder = false;

    }
```

## 401-600 解释

首先，这段代码展示了一些在 Vue.js 中广泛使用的实用函数和变量声明。其中包括：

- `getTagNamespace`：获取元素的命名空间
- `parsePlatformTagName`：解析特定平台的标签名称
- `mustUseProp`：检查属性是否必须使用属性绑定
- `async`：指示是否异步执行更新操作
- `_lifecycleHooks`：已暴露出来的生命周期钩子数组

接下来是一些辅助函数：

- `unicodeRegExp`：用于解析 HTML 标记、组件名称和属性路径的 Unicode 字符集正则表达式
- `isReserved`：判断一个字符串是否以$或\_开头，用来判断是否为保留字符
- `def`：定义一个对象属性
- `parsePath`：简单路径解析

最后，这段代码还包括了一些常用的浏览器环境和功能检测，例如检测是否支持 Passive 事件监听、是否存在 Vue 开发者工具等，以及虚拟节点类的定义。

总之，这段代码提供了 Vue.js 中许多重要的实用函数和变量声明，以及一些常用的浏览器环境和功能检测。

# 600~840

```js
    Object.defineProperty(VNode.prototype, "child", {

        // DEPRECATED: alias for componentInstance for backwards compat.

        /* istanbul ignore next */

        get: function () {

            return this.componentInstance;

        },

        enumerable: false,

        configurable: true

    });

    return VNode;

}());

var createEmptyVNode = function (text) {

    if (text === void 0) { text = ''; }

    var node = new VNode();

    node.text = text;

    node.isComment = true;

    return node;

};

function createTextVNode(val) {

    return new VNode(undefined, undefined, undefined, String(val));

}

// optimized shallow clone

// used for static nodes and slot nodes because they may be reused across

// multiple renders, cloning them avoids errors when DOM manipulations rely

// on their elm reference.

function cloneVNode(vnode) {

    var cloned = new VNode(vnode.tag, vnode.data,

    // #7975

    // clone children array to avoid mutating original in case of cloning

    // a child.

    vnode.children && vnode.children.slice(), vnode.text, vnode.elm, vnode.context, vnode.componentOptions, vnode.asyncFactory);

    cloned.ns = vnode.ns;

    cloned.isStatic = vnode.isStatic;

    cloned.key = vnode.key;

    cloned.isComment = vnode.isComment;

    cloned.fnContext = vnode.fnContext;

    cloned.fnOptions = vnode.fnOptions;

    cloned.fnScopeId = vnode.fnScopeId;

    cloned.asyncMeta = vnode.asyncMeta;

    cloned.isCloned = true;

    return cloned;

}



/* not type checking this file because flow doesn't play well with Proxy */

var initProxy;

if (process.env.NODE_ENV !== 'production') {

    var allowedGlobals_1 = makeMap('Infinity,undefined,NaN,isFinite,isNaN,' +

        'parseFloat,parseInt,decodeURI,decodeURIComponent,encodeURI,encodeURIComponent,' +

        'Math,Number,Date,Array,Object,Boolean,String,RegExp,Map,Set,JSON,Intl,BigInt,' +

        'require' // for Webpack/Browserify

    );

    var warnNonPresent_1 = function (target, key) {

        warn$2("Property or method \"".concat(key, "\" is not defined on the instance but ") +

            'referenced during render. Make sure that this property is reactive, ' +

            'either in the data option, or for class-based components, by ' +

            'initializing the property. ' +

            'See: https://v2.vuejs.org/v2/guide/reactivity.html#Declaring-Reactive-Properties.', target);

    };

    var warnReservedPrefix_1 = function (target, key) {

        warn$2("Property \"".concat(key, "\" must be accessed with \"$data.").concat(key, "\" because ") +

            'properties starting with "$" or "_" are not proxied in the Vue instance to ' +

            'prevent conflicts with Vue internals. ' +

            'See: https://v2.vuejs.org/v2/api/#data', target);

    };

    var hasProxy_1 = typeof Proxy !== 'undefined' && isNative(Proxy);

    if (hasProxy_1) {

        var isBuiltInModifier_1 = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact');

        config.keyCodes = new Proxy(config.keyCodes, {

            set: function (target, key, value) {

                if (isBuiltInModifier_1(key)) {

                    warn$2("Avoid overwriting built-in modifier in config.keyCodes: .".concat(key));

                    return false;

                }

                else {

                    target[key] = value;

                    return true;

                }

            }

        });

    }

    var hasHandler_1 = {

        has: function (target, key) {

            var has = key in target;

            var isAllowed = allowedGlobals_1(key) ||

                (typeof key === 'string' &&

                    key.charAt(0) === '_' &&

                    !(key in target.$data));

            if (!has && !isAllowed) {

                if (key in target.$data)

                    warnReservedPrefix_1(target, key);

                else

                    warnNonPresent_1(target, key);

            }

            return has || !isAllowed;

        }

    };

    var getHandler_1 = {

        get: function (target, key) {

            if (typeof key === 'string' && !(key in target)) {

                if (key in target.$data)

                    warnReservedPrefix_1(target, key);

                else

                    warnNonPresent_1(target, key);

            }

            return target[key];

        }

    };

    initProxy = function initProxy(vm) {

        if (hasProxy_1) {

            // determine which proxy handler to use

            var options = vm.$options;

            var handlers = options.render && options.render._withStripped ? getHandler_1 : hasHandler_1;

            vm._renderProxy = new Proxy(vm, handlers);

        }

        else {

            vm._renderProxy = vm;

        }

    };

}



/******************************************************************************

Copyright (c) Microsoft Corporation.



Permission to use, copy, modify, and/or distribute this software for any

purpose with or without fee is hereby granted.



THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH

REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY

AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,

INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM

LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR

OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR

PERFORMANCE OF THIS SOFTWARE.

***************************************************************************** */



var __assign = function() {

    __assign = Object.assign || function __assign(t) {

        for (var s, i = 1, n = arguments.length; i < n; i++) {

            s = arguments[i];

            for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p)) t[p] = s[p];

        }

        return t;

    };

    return __assign.apply(this, arguments);

};



var uid$2 = 0;

var pendingCleanupDeps = [];

var cleanupDeps = function () {

    for (var i = 0; i < pendingCleanupDeps.length; i++) {

        var dep = pendingCleanupDeps[i];

        dep.subs = dep.subs.filter(function (s) { return s; });

        dep._pending = false;

    }

    pendingCleanupDeps.length = 0;

};

/**

 * A dep is an observable that can have multiple

 * directives subscribing to it.

 * @internal

 */

var Dep = /** @class */ (function () {

    function Dep() {

        // pending subs cleanup

        this._pending = false;

        this.id = uid$2++;

        this.subs = [];

    }

    Dep.prototype.addSub = function (sub) {

        this.subs.push(sub);

    };

    Dep.prototype.removeSub = function (sub) {

        // #12696 deps with massive amount of subscribers are extremely slow to

        // clean up in Chromium

        // to workaround this, we unset the sub for now, and clear them on

        // next scheduler flush.

        this.subs[this.subs.indexOf(sub)] = null;

        if (!this._pending) {

            this._pending = true;

            pendingCleanupDeps.push(this);

        }

    };

    Dep.prototype.depend = function (info) {

        if (Dep.target) {

            Dep.target.addDep(this);

            if (process.env.NODE_ENV !== 'production' && info && Dep.target.onTrack) {

                Dep.target.onTrack(__assign({ effect: Dep.target }, info));

            }

        }

    };

    Dep.prototype.notify = function (info) {

        // stabilize the subscriber list first

        var subs = this.subs.filter(function (s) { return s; });

        if (process.env.NODE_ENV !== 'production' && !config.async) {

            // subs aren't sorted in scheduler if not running async

            // we need to sort them now to make sure they fire in correct

            // order

            subs.sort(function (a, b) { return a.id - b.id; });

        }

        for (var i = 0, l = subs.length; i < l; i++) {

            var sub = subs[i];

            if (process.env.NODE_ENV !== 'production' && info) {

                sub.onTrigger &&

                    sub.onTrigger(__assign({ effect: subs[i] }, info));

            }

            sub.update();

        }

    };

    return Dep;

}());

// The current target watcher being evaluated.

// This is globally unique because only one watcher

// can be evaluated at a time.

Dep.target = null;

var targetStack = [];

function pushTarget(target) {

    targetStack.push(target);

    Dep.target = target;

}

function popTarget() {

    targetStack.pop();

    Dep.target = targetStack[targetStack.length - 1];

}



/*

 * not type checking this file because flow doesn't play well with

 * dynamically accessing methods on Array prototype

 */

var arrayProto = Array.prototype;

var arrayMethods = Object.create(arrayProto);

var methodsToPatch = [

    'push',

    'pop',

    'shift',

    'unshift',

    'splice',

    'sort',

    'reverse'

]
```

## 601-840 解释

这段代码包括了 Vue.js 中的一些核心逻辑和工具函数，主要包括以下几个部分：

1.  虚拟节点对象 VNode 的定义和相关函数的实现

- `VNode`：虚拟节点构造函数
- `createEmptyVNode`：用于创建空虚拟节点的函数
- `createTextVNode`：用于创建文本节点的函数
- `cloneVNode`：用于优化浅克隆的函数

2.  Vue.js 开发调试工具的实现

- 通过设置 Proxy 代理实例来检测访问不存在属性或方法，并给予警告
- 该功能在生产环境下被禁用

3.  响应式系统的基石 Dep 类

- 管理依赖关系和通知更新
- 包括添加和删除依赖项、依赖项收集、通知依赖项更新等核心操作
- 全局唯一的 target watcher

4.  对 JavaScript 数组原型方法进行代理的实现

- 使得在修改响应式数组时能够正确地触发视图更新
  总体而言，这段代码展示了 Vue.js 中一些非常重要的核心功能和工具函数的实现方式。

# 841~1054

```js
/**

 * Intercept mutating methods and emit events

 */

   methodsToPatch.forEach(function (method) {

    // cache original method

    var original = arrayProto[method];

    def(arrayMethods, method, function mutator() {

        var args = [];

        for (var _i = 0; _i < arguments.length; _i++) {

            args[_i] = arguments[_i];

        }

        var result = original.apply(this, args);

        var ob = this.__ob__;

        var inserted;

        switch (method) {

            case 'push':

            case 'unshift':

                inserted = args;

                break;

            case 'splice':

                inserted = args.slice(2);

                break;

        }

        if (inserted)

            ob.observeArray(inserted);

        // notify change

        if (process.env.NODE_ENV !== 'production') {

            ob.dep.notify({

                type: "array mutation" /* TriggerOpTypes.ARRAY_MUTATION */,

                target: this,

                key: method

            });

        }

        else {

            ob.dep.notify();

        }

        return result;

    });

});



var arrayKeys = Object.getOwnPropertyNames(arrayMethods);

var NO_INIITIAL_VALUE = {};

/**

 * In some cases we may want to disable observation inside a component's

 * update computation.

 */

var shouldObserve = true;

function toggleObserving(value) {

    shouldObserve = value;

}

// ssr mock dep

var mockDep = {

    notify: noop,

    depend: noop,

    addSub: noop,

    removeSub: noop

};

/**

 * Observer class that is attached to each observed

 * object. Once attached, the observer converts the target

 * object's property keys into getter/setters that

 * collect dependencies and dispatch updates.

 */

var Observer = /** @class */ (function () {

    function Observer(value, shallow, mock) {

        if (shallow === void 0) { shallow = false; }

        if (mock === void 0) { mock = false; }

        this.value = value;

        this.shallow = shallow;

        this.mock = mock;

        // this.value = value

        this.dep = mock ? mockDep : new Dep();

        this.vmCount = 0;

        def(value, '__ob__', this);

        if (isArray(value)) {

            if (!mock) {

                if (hasProto) {

                    value.__proto__ = arrayMethods;

                    /* eslint-enable no-proto */

                }

                else {

                    for (var i = 0, l = arrayKeys.length; i < l; i++) {

                        var key = arrayKeys[i];

                        def(value, key, arrayMethods[key]);

                    }

                }

            }

            if (!shallow) {

                this.observeArray(value);

            }

        }

        else {

            /**

             * Walk through all properties and convert them into

             * getter/setters. This method should only be called when

             * value type is Object.

             */

            var keys = Object.keys(value);

            for (var i = 0; i < keys.length; i++) {

                var key = keys[i];

                defineReactive(value, key, NO_INIITIAL_VALUE, undefined, shallow, mock);

            }

        }

    }

    /**

     * Observe a list of Array items.

     */

    Observer.prototype.observeArray = function (value) {

        for (var i = 0, l = value.length; i < l; i++) {

            observe(value[i], false, this.mock);

        }

    };

    return Observer;

}());

// helpers

/**

 * Attempt to create an observer instance for a value,

 * returns the new observer if successfully observed,

 * or the existing observer if the value already has one.

 */

function observe(value, shallow, ssrMockReactivity) {

    if (value && hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {

        return value.__ob__;

    }

    if (shouldObserve &&

        (ssrMockReactivity || !isServerRendering()) &&

        (isArray(value) || isPlainObject(value)) &&

        Object.isExtensible(value) &&

        !value.__v_skip /* ReactiveFlags.SKIP */ &&

        !isRef(value) &&

        !(value instanceof VNode)) {

        return new Observer(value, shallow, ssrMockReactivity);

    }

}

/**

 * Define a reactive property on an Object.

 */

function defineReactive(obj, key, val, customSetter, shallow, mock) {

    var dep = new Dep();

    var property = Object.getOwnPropertyDescriptor(obj, key);

    if (property && property.configurable === false) {

        return;

    }

    // cater for pre-defined getter/setters

    var getter = property && property.get;

    var setter = property && property.set;

    if ((!getter || setter) &&

        (val === NO_INIITIAL_VALUE || arguments.length === 2)) {

        val = obj[key];

    }

    var childOb = !shallow && observe(val, false, mock);

    Object.defineProperty(obj, key, {

        enumerable: true,

        configurable: true,

        get: function reactiveGetter() {

            var value = getter ? getter.call(obj) : val;

            if (Dep.target) {

                if (process.env.NODE_ENV !== 'production') {

                    dep.depend({

                        target: obj,

                        type: "get" /* TrackOpTypes.GET */,

                        key: key

                    });

                }

                else {

                    dep.depend();

                }

                if (childOb) {

                    childOb.dep.depend();

                    if (isArray(value)) {

                        dependArray(value);

                    }

                }

            }

            return isRef(value) && !shallow ? value.value : value;

        },

        set: function reactiveSetter(newVal) {

            var value = getter ? getter.call(obj) : val;

            if (!hasChanged(value, newVal)) {

                return;

            }

            if (process.env.NODE_ENV !== 'production' && customSetter) {

                customSetter();

            }

            if (setter) {

                setter.call(obj, newVal);

            }

            else if (getter) {

                // #7981: for accessor properties without setter

                return;

            }

            else if (!shallow && isRef(value) && !isRef(newVal)) {

                value.value = newVal;

                return;

            }

            else {

                val = newVal;

            }

            childOb = !shallow && observe(newVal, false, mock);

            if (process.env.NODE_ENV !== 'production') {

                dep.notify({

                    type: "set" /* TriggerOpTypes.SET */,

                    target: obj,

                    key: key,

                    newValue: newVal,

                    oldValue: value

                });

            }

            else {

                dep.notify();

            }

        }

    });

    return dep;
```

## 841-1044 解释

这段代码实现了 Vue.js 响应式系统的核心逻辑，包括 Observer 类和 defineReactive 函数。Observer 类是一个观察者，将目标对象转换为可监听的对象，并将目标对象的所有属性变成 getter/setter 形式。defineReactive 函数则用于定义一个新属性或修改已有属性，使其成为响应式属性。

具体来说，defineReactive 函数通过 Object.defineProperty()方法将属性定义为可监听的。当属性被读取时，会收集依赖关系并在属性被修改时通知更新。同时，如果属性值是一个对象，则会对其进行递归处理，使其成为嵌套的响应式对象。

Observer 类则用于管理依赖项，它会为每个被监听的对象创建一个 Dep 实例，并在目标对象属性的 getter 方法中收集依赖项，在 setter 方法中通知这些依赖项进行更新。同时，如果目标对象是数组，则会为其中每个元素也创建一个 Observer 实例，使得数组也可以响应式地更新。

总之，这段代码展示了 Vue.js 响应式系统的重要实现方式，是 Vue.js 能够实现数据驱动视图更新的核心。

# 1055~1226

```js
function set(target, key, val) {
  if (
    process.env.NODE_ENV !== "production" &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn$2(
      "Cannot set reactive property on undefined, null, or primitive value: ".concat(
        target
      )
    );
  }

  if (isReadonly(target)) {
    process.env.NODE_ENV !== "production" &&
      warn$2(
        'Set operation on key "'.concat(key, '" failed: target is readonly.')
      );

    return;
  }

  var ob = target.__ob__;

  if (isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);

    target.splice(key, 1, val); // when mocking for SSR, array methods are not hijacked

    if (ob && !ob.shallow && ob.mock) {
      observe(val, false, true);
    }

    return val;
  }

  if (key in target && !(key in Object.prototype)) {
    target[key] = val;

    return val;
  }

  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== "production" &&
      warn$2(
        "Avoid adding reactive properties to a Vue instance or its root $data " +
          "at runtime - declare it upfront in the data option."
      );

    return val;
  }

  if (!ob) {
    target[key] = val;

    return val;
  }

  defineReactive(ob.value, key, val, undefined, ob.shallow, ob.mock);

  if (process.env.NODE_ENV !== "production") {
    ob.dep.notify({
      type: "add" /* TriggerOpTypes.ADD */,

      target: target,

      key: key,

      newValue: val,

      oldValue: undefined,
    });
  } else {
    ob.dep.notify();
  }

  return val;
}

function del(target, key) {
  if (
    process.env.NODE_ENV !== "production" &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn$2(
      "Cannot delete reactive property on undefined, null, or primitive value: ".concat(
        target
      )
    );
  }

  if (isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1);

    return;
  }

  var ob = target.__ob__;

  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== "production" &&
      warn$2(
        "Avoid deleting properties on a Vue instance or its root $data " +
          "- just set it to null."
      );

    return;
  }

  if (isReadonly(target)) {
    process.env.NODE_ENV !== "production" &&
      warn$2(
        'Delete operation on key "'.concat(key, '" failed: target is readonly.')
      );

    return;
  }

  if (!hasOwn(target, key)) {
    return;
  }

  delete target[key];

  if (!ob) {
    return;
  }

  if (process.env.NODE_ENV !== "production") {
    ob.dep.notify({
      type: "delete" /* TriggerOpTypes.DELETE */,

      target: target,

      key: key,
    });
  } else {
    ob.dep.notify();
  }
}

/**

 * Collect dependencies on array elements when the array is touched, since

 * we cannot intercept array element access like property getters.

 */

function dependArray(value) {
  for (var e = void 0, i = 0, l = value.length; i < l; i++) {
    e = value[i];

    if (e && e.__ob__) {
      e.__ob__.dep.depend();
    }

    if (isArray(e)) {
      dependArray(e);
    }
  }
}

function reactive(target) {
  makeReactive(target, false);

  return target;
}

/**

 * Return a shallowly-reactive copy of the original object, where only the root

 * level properties are reactive. It also does not auto-unwrap refs (even at the

 * root level).

 */

function shallowReactive(target) {
  makeReactive(target, true);

  def(target, "__v_isShallow" /* ReactiveFlags.IS_SHALLOW */, true);

  return target;
}

function makeReactive(target, shallow) {
  // if trying to observe a readonly proxy, return the readonly version.

  if (!isReadonly(target)) {
    if (process.env.NODE_ENV !== "production") {
      if (isArray(target)) {
        warn$2(
          "Avoid using Array as root value for "
            .concat(
              shallow ? "shallowReactive()" : "reactive()",
              " as it cannot be tracked in watch() or watchEffect(). Use "
            )
            .concat(
              shallow ? "shallowRef()" : "ref()",
              " instead. This is a Vue-2-only limitation."
            )
        );
      }

      var existingOb = target && target.__ob__;

      if (existingOb && existingOb.shallow !== shallow) {
        warn$2(
          "Target is already a "
            .concat(
              existingOb.shallow ? "" : "non-",
              "shallow reactive object, and cannot be converted to "
            )
            .concat(shallow ? "" : "non-", "shallow.")
        );
      }
    }

    var ob = observe(
      target,
      shallow,
      isServerRendering() /* ssr mock reactivity */
    );

    if (process.env.NODE_ENV !== "production" && !ob) {
      if (target == null || isPrimitive(target)) {
        warn$2("value cannot be made reactive: ".concat(String(target)));
      }

      if (isCollectionType(target)) {
        warn$2(
          "Vue 2 does not support reactive collection types such as Map or Set."
        );
      }
    }
  }
}

function isReactive(value) {
  if (isReadonly(value)) {
    return isReactive(value["__v_raw" /* ReactiveFlags.RAW */]);
  }

  return !!(value && value.__ob__);
}

function isShallow(value) {
  return !!(value && value.__v_isShallow);
}

function isReadonly(value) {
  return !!(value && value.__v_isReadonly);
}

function isProxy(value) {
  return isReactive(value) || isReadonly(value);
}

function toRaw(observed) {
  var raw = observed && observed["__v_raw" /* ReactiveFlags.RAW */];

  return raw ? toRaw(raw) : observed;
}

function markRaw(value) {
  // non-extensible objects won't be observed anyway

  if (Object.isExtensible(value)) {
    def(value, "__v_skip" /* ReactiveFlags.SKIP */, true);
  }

  return value;
}

/**

 * @internal

 */

function isCollectionType(value) {
  var type = toRawType(value);

  return (
    type === "Map" || type === "WeakMap" || type === "Set" || type === "WeakSet"
  );
}
```

## 1045- 1205 解释

这段代码实现了 Vue.js 响应式系统的核心逻辑，包括数据劫持和依赖收集。通过将对象转换为可监听的对象，使得当对象属性被修改时，能够自动触发更新视图等操作。具体实现方式包括：使用 Observer 类将目标对象转换为可监听的对象，并使用 defineReactive 函数定义 getter/setter 方法；使用 Dep 类进行依赖管理，使用 set 函数添加或更新属性值、使用 del 函数删除属性值、使用 dependArray 函数收集数组元素的依赖关系。同时还定义了一些辅助函数如 reactive、shallowReactive、isReactive、isShallow、isReadonly、isProxy、toRaw、markRaw 等，用于判断对象是否可响应、获取原始对象等操作。总之，这段代码是 Vue.js 响应式系统的核心实现，是 Vue.js 实现数据驱动的重要基础。

# 1206~1501

```js
/**

 * @internal

 */

var RefFlag = "__v_isRef";

function isRef(r) {
  return !!(r && r.__v_isRef === true);
}

function ref$1(value) {
  return createRef(value, false);
}

function shallowRef(value) {
  return createRef(value, true);
}

function createRef(rawValue, shallow) {
  if (isRef(rawValue)) {
    return rawValue;
  }

  var ref = {};

  def(ref, RefFlag, true);

  def(ref, "__v_isShallow" /* ReactiveFlags.IS_SHALLOW */, shallow);

  def(
    ref,
    "dep",
    defineReactive(ref, "value", rawValue, null, shallow, isServerRendering())
  );

  return ref;
}

function triggerRef(ref) {
  if (process.env.NODE_ENV !== "production" && !ref.dep) {
    warn$2("received object is not a triggerable ref.");
  }

  if (process.env.NODE_ENV !== "production") {
    ref.dep &&
      ref.dep.notify({
        type: "set" /* TriggerOpTypes.SET */,

        target: ref,

        key: "value",
      });
  } else {
    ref.dep && ref.dep.notify();
  }
}

function unref(ref) {
  return isRef(ref) ? ref.value : ref;
}

function proxyRefs(objectWithRefs) {
  if (isReactive(objectWithRefs)) {
    return objectWithRefs;
  }

  var proxy = {};

  var keys = Object.keys(objectWithRefs);

  for (var i = 0; i < keys.length; i++) {
    proxyWithRefUnwrap(proxy, objectWithRefs, keys[i]);
  }

  return proxy;
}

function proxyWithRefUnwrap(target, source, key) {
  Object.defineProperty(target, key, {
    enumerable: true,

    configurable: true,

    get: function () {
      var val = source[key];

      if (isRef(val)) {
        return val.value;
      } else {
        var ob = val && val.__ob__;

        if (ob) ob.dep.depend();

        return val;
      }
    },

    set: function (value) {
      var oldValue = source[key];

      if (isRef(oldValue) && !isRef(value)) {
        oldValue.value = value;
      } else {
        source[key] = value;
      }
    },
  });
}

function customRef(factory) {
  var dep = new Dep();

  var _a = factory(
      function () {
        if (process.env.NODE_ENV !== "production") {
          dep.depend({
            target: ref,

            type: "get" /* TrackOpTypes.GET */,

            key: "value",
          });
        } else {
          dep.depend();
        }
      },
      function () {
        if (process.env.NODE_ENV !== "production") {
          dep.notify({
            target: ref,

            type: "set" /* TriggerOpTypes.SET */,

            key: "value",
          });
        } else {
          dep.notify();
        }
      }
    ),
    get = _a.get,
    set = _a.set;

  var ref = {
    get value() {
      return get();
    },

    set value(newVal) {
      set(newVal);
    },
  };

  def(ref, RefFlag, true);

  return ref;
}

function toRefs(object) {
  if (process.env.NODE_ENV !== "production" && !isReactive(object)) {
    warn$2("toRefs() expects a reactive object but received a plain one.");
  }

  var ret = isArray(object) ? new Array(object.length) : {};

  for (var key in object) {
    ret[key] = toRef(object, key);
  }

  return ret;
}

function toRef(object, key, defaultValue) {
  var val = object[key];

  if (isRef(val)) {
    return val;
  }

  var ref = {
    get value() {
      var val = object[key];

      return val === undefined ? defaultValue : val;
    },

    set value(newVal) {
      object[key] = newVal;
    },
  };

  def(ref, RefFlag, true);

  return ref;
}

var rawToReadonlyFlag = "__v_rawToReadonly";

var rawToShallowReadonlyFlag = "__v_rawToShallowReadonly";

function readonly(target) {
  return createReadonly(target, false);
}

function createReadonly(target, shallow) {
  if (!isPlainObject(target)) {
    if (process.env.NODE_ENV !== "production") {
      if (isArray(target)) {
        warn$2("Vue 2 does not support readonly arrays.");
      } else if (isCollectionType(target)) {
        warn$2(
          "Vue 2 does not support readonly collection types such as Map or Set."
        );
      } else {
        warn$2("value cannot be made readonly: ".concat(typeof target));
      }
    }

    return target;
  }

  if (process.env.NODE_ENV !== "production" && !Object.isExtensible(target)) {
    warn$2(
      "Vue 2 does not support creating readonly proxy for non-extensible object."
    );
  } // already a readonly object

  if (isReadonly(target)) {
    return target;
  } // already has a readonly proxy

  var existingFlag = shallow ? rawToShallowReadonlyFlag : rawToReadonlyFlag;

  var existingProxy = target[existingFlag];

  if (existingProxy) {
    return existingProxy;
  }

  var proxy = Object.create(Object.getPrototypeOf(target));

  def(target, existingFlag, proxy);

  def(proxy, "__v_isReadonly" /* ReactiveFlags.IS_READONLY */, true);

  def(proxy, "__v_raw" /* ReactiveFlags.RAW */, target);

  if (isRef(target)) {
    def(proxy, RefFlag, true);
  }

  if (shallow || isShallow(target)) {
    def(proxy, "__v_isShallow" /* ReactiveFlags.IS_SHALLOW */, true);
  }

  var keys = Object.keys(target);

  for (var i = 0; i < keys.length; i++) {
    defineReadonlyProperty(proxy, target, keys[i], shallow);
  }

  return proxy;
}

function defineReadonlyProperty(proxy, target, key, shallow) {
  Object.defineProperty(proxy, key, {
    enumerable: true,

    configurable: true,

    get: function () {
      var val = target[key];

      return shallow || !isPlainObject(val) ? val : readonly(val);
    },

    set: function () {
      process.env.NODE_ENV !== "production" &&
        warn$2(
          'Set operation on key "'.concat(key, '" failed: target is readonly.')
        );
    },
  });
}

/**

 * Returns a reactive-copy of the original object, where only the root level

 * properties are readonly, and does NOT unwrap refs nor recursively convert

 * returned properties.

 * This is used for creating the props proxy object for stateful components.

 */

function shallowReadonly(target) {
  return createReadonly(target, true);
}

function computed(getterOrOptions, debugOptions) {
  var getter;

  var setter;

  var onlyGetter = isFunction(getterOrOptions);

  if (onlyGetter) {
    getter = getterOrOptions;

    setter =
      process.env.NODE_ENV !== "production"
        ? function () {
            warn$2("Write operation failed: computed value is readonly");
          }
        : noop;
  } else {
    getter = getterOrOptions.get;

    setter = getterOrOptions.set;
  }

  var watcher = isServerRendering()
    ? null
    : new Watcher(currentInstance, getter, noop, { lazy: true });

  if (process.env.NODE_ENV !== "production" && watcher && debugOptions) {
    watcher.onTrack = debugOptions.onTrack;

    watcher.onTrigger = debugOptions.onTrigger;
  }

  var ref = {
    // some libs rely on the presence effect for checking computed refs

    // from normal refs, but the implementation doesn't matter

    effect: watcher,

    get value() {
      if (watcher) {
        if (watcher.dirty) {
          watcher.evaluate();
        }

        if (Dep.target) {
          if (process.env.NODE_ENV !== "production" && Dep.target.onTrack) {
            Dep.target.onTrack({
              effect: Dep.target,

              target: ref,

              type: "get" /* TrackOpTypes.GET */,

              key: "value",
            });
          }

          watcher.depend();
        }

        return watcher.value;
      } else {
        return getter();
      }
    },

    set value(newVal) {
      setter(newVal);
    },
  };

  def(ref, RefFlag, true);

  def(ref, "__v_isReadonly" /* ReactiveFlags.IS_READONLY */, onlyGetter);

  return ref;
}
```

## 1206-1501 解释

这段代码实现了 Vue 2.x 中的响应式系统，包括了 ref、shallowRef、triggerRef、unref、proxyRefs、customRef、toRefs、readonly、shallowReadonly 和 computed 等 API。其中，ref 和 shallowRef 可以用来创建响应式数据对象；triggerRef 可以触发响应式数据对象的依赖更新；unref 可以用来获取非响应式数据对象；proxyRefs 可以将带有响应式属性的对象转化为 Proxy 对象；customRef 可以创建自定义的 ref 对象；toRefs 可以将一个响应式对象转换成由其属性组成的 ref 对象集合；readonly 和 shallowReadonly 可以分别创建只读的深层次和浅层次响应式数据对象；computed 可以用来计算一个响应式数据对象的值。

# 1502~1692

```js
var mark;

var measure;

if (process.env.NODE_ENV !== "production") {
  var perf_1 = inBrowser && window.performance; /* istanbul ignore if */

  if (
    perf_1 && // @ts-ignore
    perf_1.mark && // @ts-ignore
    perf_1.measure && // @ts-ignore
    perf_1.clearMarks && // @ts-ignore
    perf_1.clearMeasures
  ) {
    mark = function (tag) {
      return perf_1.mark(tag);
    };

    measure = function (name, startTag, endTag) {
      perf_1.measure(name, startTag, endTag);

      perf_1.clearMarks(startTag);

      perf_1.clearMarks(endTag); // perf.clearMeasures(name)
    };
  }
}

var normalizeEvent = cached(function (name) {
  var passive = name.charAt(0) === "&";

  name = passive ? name.slice(1) : name;

  var once = name.charAt(0) === "~"; // Prefixed last, checked first

  name = once ? name.slice(1) : name;

  var capture = name.charAt(0) === "!";

  name = capture ? name.slice(1) : name;

  return {
    name: name,

    once: once,

    capture: capture,

    passive: passive,
  };
});

function createFnInvoker(fns, vm) {
  function invoker() {
    var fns = invoker.fns;

    if (isArray(fns)) {
      var cloned = fns.slice();

      for (var i = 0; i < cloned.length; i++) {
        invokeWithErrorHandling(cloned[i], null, arguments, vm, "v-on handler");
      }
    } else {
      // return handler return value for single handlers

      return invokeWithErrorHandling(fns, null, arguments, vm, "v-on handler");
    }
  }

  invoker.fns = fns;

  return invoker;
}

function updateListeners(on, oldOn, add, remove, createOnceHandler, vm) {
  var name, cur, old, event;

  for (name in on) {
    cur = on[name];

    old = oldOn[name];

    event = normalizeEvent(name);

    if (isUndef(cur)) {
      process.env.NODE_ENV !== "production" &&
        warn$2(
          'Invalid handler for event "'.concat(event.name, '": got ') +
            String(cur),
          vm
        );
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm);
      }

      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture);
      }

      add(event.name, cur, event.capture, event.passive, event.params);
    } else if (cur !== old) {
      old.fns = cur;

      on[name] = old;
    }
  }

  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name);

      remove(event.name, oldOn[name], event.capture);
    }
  }
}

function mergeVNodeHook(def, hookKey, hook) {
  if (def instanceof VNode) {
    def = def.data.hook || (def.data.hook = {});
  }

  var invoker;

  var oldHook = def[hookKey];

  function wrappedHook() {
    hook.apply(this, arguments); // important: remove merged hook to ensure it's called only once // and prevent memory leak

    remove$2(invoker.fns, wrappedHook);
  }

  if (isUndef(oldHook)) {
    // no existing hook

    invoker = createFnInvoker([wrappedHook]);
  } else {
    /* istanbul ignore if */

    if (isDef(oldHook.fns) && isTrue(oldHook.merged)) {
      // already a merged invoker

      invoker = oldHook;

      invoker.fns.push(wrappedHook);
    } else {
      // existing plain hook

      invoker = createFnInvoker([oldHook, wrappedHook]);
    }
  }

  invoker.merged = true;

  def[hookKey] = invoker;
}

function extractPropsFromVNodeData(data, Ctor, tag) {
  // we are only extracting raw values here.

  // validation and default values are handled in the child

  // component itself.

  var propOptions = Ctor.options.props;

  if (isUndef(propOptions)) {
    return;
  }

  var res = {};

  var attrs = data.attrs,
    props = data.props;

  if (isDef(attrs) || isDef(props)) {
    for (var key in propOptions) {
      var altKey = hyphenate(key);

      if (process.env.NODE_ENV !== "production") {
        var keyInLowerCase = key.toLowerCase();

        if (key !== keyInLowerCase && attrs && hasOwn(attrs, keyInLowerCase)) {
          tip(
            'Prop "'.concat(keyInLowerCase, '" is passed to component ') +
              "".concat(
                formatComponentName(
                  // @ts-expect-error tag is string

                  tag || Ctor
                ),
                ", but the declared prop name is"
              ) +
              ' "'.concat(key, '". ') +
              "Note that HTML attributes are case-insensitive and camelCased " +
              "props need to use their kebab-case equivalents when using in-DOM " +
              'templates. You should probably use "'
                .concat(altKey, '" instead of "')
                .concat(key, '".')
          );
        }
      }

      checkProp(res, props, key, altKey, true) ||
        checkProp(res, attrs, key, altKey, false);
    }
  }

  return res;
}

function checkProp(res, hash, key, altKey, preserve) {
  if (isDef(hash)) {
    if (hasOwn(hash, key)) {
      res[key] = hash[key];

      if (!preserve) {
        delete hash[key];
      }

      return true;
    } else if (hasOwn(hash, altKey)) {
      res[key] = hash[altKey];

      if (!preserve) {
        delete hash[altKey];
      }

      return true;
    }
  }

  return false;
}

// The template compiler attempts to minimize the need for normalization by

// statically analyzing the template at compile time.

//

// For plain HTML markup, normalization can be completely skipped because the

// generated render function is guaranteed to return Array<VNode>. There are

// two cases where extra normalization is needed:

// 1. When the children contains components - because a functional component

// may return an Array instead of a single root. In this case, just a simple

// normalization is needed - if any child is an Array, we flatten the whole

// thing with Array.prototype.concat. It is guaranteed to be only 1-level deep

// because functional components already normalize their own children.

function simpleNormalizeChildren(children) {
  for (var i = 0; i < children.length; i++) {
    if (isArray(children[i])) {
      return Array.prototype.concat.apply([], children);
    }
  }

  return children;
}
```

## 1502~1692 解释

这段代码是与 Vue.js 或其他前端 JavaScript 框架相关的实用函数集合。

代码的第一部分检查当前环境中是否存在某些 API，并在使用这些 API 之前进行判断，例如 `performance` API。

代码的第二部分包括处理事件监听器、合并 VNode 钩子、从 VNode 数据中提取 props 和规范化 children 的函数。

`updateListeners` 函数根据传入的 `on` 对象的更改更新一个 DOM 节点上的事件监听器；它添加新的事件监听器、删除旧的监听器和合并现有的监听器。

`mergeVNodeHook` 函数通过创建一个新的调用两个钩子的调用者函数来合并两个 VNode 钩子。

`extractPropsFromVNodeData` 函数根据组件选项中的 prop 定义从 VNode 的 `data` 对象中提取 props。

最后，`simpleNormalizeChildren` 函数通过将任何嵌套数组展平成单层数组来规范化子元素的列表。

# 1692~1953

```js
function normalizeChildren(children) {
  return isPrimitive(children)
    ? [createTextVNode(children)]
    : isArray(children)
    ? normalizeArrayChildren(children)
    : undefined;
}

function isTextNode(node) {
  return isDef(node) && isDef(node.text) && isFalse(node.isComment);
}

function normalizeArrayChildren(children, nestedIndex) {
  var res = [];

  var i, c, lastIndex, last;

  for (i = 0; i < children.length; i++) {
    c = children[i];

    if (isUndef(c) || typeof c === "boolean") continue;

    lastIndex = res.length - 1;

    last = res[lastIndex]; //  nested

    if (isArray(c)) {
      if (c.length > 0) {
        c = normalizeArrayChildren(
          c,
          "".concat(nestedIndex || "", "_").concat(i)
        ); // merge adjacent text nodes

        if (isTextNode(c[0]) && isTextNode(last)) {
          res[lastIndex] = createTextVNode(last.text + c[0].text);

          c.shift();
        }

        res.push.apply(res, c);
      }
    } else if (isPrimitive(c)) {
      if (isTextNode(last)) {
        // merge adjacent text nodes

        // this is necessary for SSR hydration because text nodes are

        // essentially merged when rendered to HTML strings

        res[lastIndex] = createTextVNode(last.text + c);
      } else if (c !== "") {
        // convert primitive to vnode

        res.push(createTextVNode(c));
      }
    } else {
      if (isTextNode(c) && isTextNode(last)) {
        // merge adjacent text nodes

        res[lastIndex] = createTextVNode(last.text + c.text);
      } else {
        // default key for nested array children (likely generated by v-for)

        if (
          isTrue(children._isVList) &&
          isDef(c.tag) &&
          isUndef(c.key) &&
          isDef(nestedIndex)
        ) {
          c.key = "__vlist".concat(nestedIndex, "_").concat(i, "__");
        }

        res.push(c);
      }
    }
  }

  return res;
}

var SIMPLE_NORMALIZE = 1;

var ALWAYS_NORMALIZE = 2;

// wrapper function for providing a more flexible interface

// without getting yelled at by flow

function createElement$1(
  context,
  tag,
  data,
  children,
  normalizationType,
  alwaysNormalize
) {
  if (isArray(data) || isPrimitive(data)) {
    normalizationType = children;

    children = data;

    data = undefined;
  }

  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE;
  }

  return _createElement(context, tag, data, children, normalizationType);
}

function _createElement(context, tag, data, children, normalizationType) {
  if (isDef(data) && isDef(data.__ob__)) {
    process.env.NODE_ENV !== "production" &&
      warn$2(
        "Avoid using observed data object as vnode data: ".concat(
          JSON.stringify(data),
          "\n"
        ) + "Always create fresh vnode data objects in each render!",
        context
      );

    return createEmptyVNode();
  } // object syntax in v-bind

  if (isDef(data) && isDef(data.is)) {
    tag = data.is;
  }

  if (!tag) {
    // in case of component :is set to falsy value

    return createEmptyVNode();
  } // warn against non-primitive key

  if (
    process.env.NODE_ENV !== "production" &&
    isDef(data) &&
    isDef(data.key) &&
    !isPrimitive(data.key)
  ) {
    warn$2(
      "Avoid using non-primitive value as key, " +
        "use string/number value instead.",
      context
    );
  } // support single function children as default scoped slot

  if (isArray(children) && isFunction(children[0])) {
    data = data || {};

    data.scopedSlots = { default: children[0] };

    children.length = 0;
  }

  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children);
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children);
  }

  var vnode, ns;

  if (typeof tag === "string") {
    var Ctor = void 0;

    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag);

    if (config.isReservedTag(tag)) {
      // platform built-in elements

      if (
        process.env.NODE_ENV !== "production" &&
        isDef(data) &&
        isDef(data.nativeOn) &&
        data.tag !== "component"
      ) {
        warn$2(
          "The .native modifier for v-on is only valid on components but it was used on <".concat(
            tag,
            ">."
          ),
          context
        );
      }

      vnode = new VNode(
        config.parsePlatformTagName(tag),
        data,
        children,
        undefined,
        undefined,
        context
      );
    } else if (
      (!data || !data.pre) &&
      isDef((Ctor = resolveAsset(context.$options, "components", tag)))
    ) {
      // component

      vnode = createComponent(Ctor, data, context, children, tag);
    } else {
      // unknown or unlisted namespaced elements

      // check at runtime because it may get assigned a namespace when its

      // parent normalizes children

      vnode = new VNode(tag, data, children, undefined, undefined, context);
    }
  } else {
    // direct component options / constructor

    vnode = createComponent(tag, data, context, children);
  }

  if (isArray(vnode)) {
    return vnode;
  } else if (isDef(vnode)) {
    if (isDef(ns)) applyNS(vnode, ns);

    if (isDef(data)) registerDeepBindings(data);

    return vnode;
  } else {
    return createEmptyVNode();
  }
}

function applyNS(vnode, ns, force) {
  vnode.ns = ns;

  if (vnode.tag === "foreignObject") {
    // use default namespace inside foreignObject

    ns = undefined;

    force = true;
  }

  if (isDef(vnode.children)) {
    for (var i = 0, l = vnode.children.length; i < l; i++) {
      var child = vnode.children[i];

      if (
        isDef(child.tag) &&
        (isUndef(child.ns) || (isTrue(force) && child.tag !== "svg"))
      ) {
        applyNS(child, ns, force);
      }
    }
  }
}

// ref #5318

// necessary to ensure parent re-render when deep bindings like :style and

// :class are used on slot nodes

function registerDeepBindings(data) {
  if (isObject(data.style)) {
    traverse(data.style);
  }

  if (isObject(data.class)) {
    traverse(data.class);
  }
}

/**

 * Runtime helper for rendering v-for lists.

 */

function renderList(val, render) {
  var ret = null,
    i,
    l,
    keys,
    key;

  if (isArray(val) || typeof val === "string") {
    ret = new Array(val.length);

    for (i = 0, l = val.length; i < l; i++) {
      ret[i] = render(val[i], i);
    }
  } else if (typeof val === "number") {
    ret = new Array(val);

    for (i = 0; i < val; i++) {
      ret[i] = render(i + 1, i);
    }
  } else if (isObject(val)) {
    if (hasSymbol && val[Symbol.iterator]) {
      ret = [];

      var iterator = val[Symbol.iterator]();

      var result = iterator.next();

      while (!result.done) {
        ret.push(render(result.value, ret.length));

        result = iterator.next();
      }
    } else {
      keys = Object.keys(val);

      ret = new Array(keys.length);

      for (i = 0, l = keys.length; i < l; i++) {
        key = keys[i];

        ret[i] = render(val[key], key, i);
      }
    }
  }

  if (!isDef(ret)) {
    ret = [];
  }

  ret._isVList = true;

  return ret;
}

/**

 * Runtime helper for rendering <slot>

 */

function renderSlot(name, fallbackRender, props, bindObject) {
  var scopedSlotFn = this.$scopedSlots[name];

  var nodes;

  if (scopedSlotFn) {
    // scoped slot

    props = props || {};

    if (bindObject) {
      if (process.env.NODE_ENV !== "production" && !isObject(bindObject)) {
        warn$2("slot v-bind without argument expects an Object", this);
      }

      props = extend(extend({}, bindObject), props);
    }

    nodes =
      scopedSlotFn(props) ||
      (isFunction(fallbackRender) ? fallbackRender() : fallbackRender);
  } else {
    nodes =
      this.$slots[name] ||
      (isFunction(fallbackRender) ? fallbackRender() : fallbackRender);
  }

  var target = props && props.slot;

  if (target) {
    return this.$createElement("template", { slot: target }, nodes);
  } else {
    return nodes;
  }
}
```

## 1692~1953 解释

这段代码是一些关于 Vue.js 的运行时帮助函数，包括规范化 children、渲染 v-for 循环列表和渲染插槽的函数。

其中 `normalizeChildren` 函数用于将 children 规范化为一个统一的数组格式，以便在 VNode 渲染时更容易处理。

`renderList` 函数用于渲染 v-for 循环列表，根据传入的值类型不同执行不同的循环操作，并返回由每次循环渲染得到的 VNode 数组。

`renderSlot` 函数用于渲染插槽，首先尝试使用作用域插槽进行渲染，若不存在则使用默认插槽进行渲染，并支持指定插槽名和绑定对象。

# 2209~2245

```js
function resolveFilter(id) {
  return resolveAsset(this.$options, "filters", id, true) || identity;
}

function isKeyNotMatch(expect, actual) {
  if (isArray(expect)) {
    return expect.indexOf(actual) === -1;
  } else {
    return expect !== actual;
  }
}

/**

 * Runtime helper for checking keyCodes from config.

 * exposed as Vue.prototype._k

 * passing in eventKeyName as last argument separately for backwards compat

 */

function checkKeyCodes(
  eventKeyCode,
  key,
  builtInKeyCode,
  eventKeyName,
  builtInKeyName
) {
  var mappedKeyCode = config.keyCodes[key] || builtInKeyCode;

  if (builtInKeyName && eventKeyName && !config.keyCodes[key]) {
    return isKeyNotMatch(builtInKeyName, eventKeyName);
  } else if (mappedKeyCode) {
    return isKeyNotMatch(mappedKeyCode, eventKeyCode);
  } else if (eventKeyName) {
    return hyphenate(eventKeyName) !== key;
  }

  return eventKeyCode === undefined;
}

/**

 * Runtime helper for merging v-bind="object" into a VNode's data.

 */

function bindObjectProps(data, tag, value, asProp, isSync) {
  if (value) {
    if (!isObject(value)) {
      process.env.NODE_ENV !== "production" &&
        warn$2(
          "v-bind without argument expects an Object or Array value",
          this
        );
    } else {
      if (isArray(value)) {
        value = toObject(value);
      }

      var hash = void 0;

      var _loop_1 = function (key) {
        if (key === "class" || key === "style" || isReservedAttribute(key)) {
          hash = data;
        } else {
          var type = data.attrs && data.attrs.type;

          hash =
            asProp || config.mustUseProp(tag, type, key)
              ? data.domProps || (data.domProps = {})
              : data.attrs || (data.attrs = {});
        }

        var camelizedKey = camelize(key);

        var hyphenatedKey = hyphenate(key);

        if (!(camelizedKey in hash) && !(hyphenatedKey in hash)) {
          hash[key] = value[key];

          if (isSync) {
            var on = data.on || (data.on = {});

            on["update:".concat(key)] = function ($event) {
              value[key] = $event;
            };
          }
        }
      };

      for (var key in value) {
        _loop_1(key);
      }
    }
  }

  return data;
}

/**

 * Runtime helper for rendering static trees.

 */

function renderStatic(index, isInFor) {
  var cached = this._staticTrees || (this._staticTrees = []);

  var tree = cached[index]; // if has already-rendered static tree and not inside v-for, // we can reuse the same tree.

  if (tree && !isInFor) {
    return tree;
  } // otherwise, render a fresh tree.

  tree = cached[index] = this.$options.staticRenderFns[index].call(
    this._renderProxy,
    this._c,
    this // for render fns generated for functional component templates
  );

  markStatic$1(tree, "__static__".concat(index), false);

  return tree;
}

/**

 * Runtime helper for v-once.

 * Effectively it means marking the node as static with a unique key.

 */

function markOnce(tree, index, key) {
  markStatic$1(
    tree,
    "__once__".concat(index).concat(key ? "_".concat(key) : ""),
    true
  );

  return tree;
}

function markStatic$1(tree, key, isOnce) {
  if (isArray(tree)) {
    for (var i = 0; i < tree.length; i++) {
      if (tree[i] && typeof tree[i] !== "string") {
        markStaticNode(tree[i], "".concat(key, "_").concat(i), isOnce);
      }
    }
  } else {
    markStaticNode(tree, key, isOnce);
  }
}

function markStaticNode(node, key, isOnce) {
  node.isStatic = true;

  node.key = key;

  node.isOnce = isOnce;
}

function bindObjectListeners(data, value) {
  if (value) {
    if (!isPlainObject(value)) {
      process.env.NODE_ENV !== "production" &&
        warn$2("v-on without argument expects an Object value", this);
    } else {
      var on = (data.on = data.on ? extend({}, data.on) : {});

      for (var key in value) {
        var existing = on[key];

        var ours = value[key];

        on[key] = existing ? [].concat(existing, ours) : ours;
      }
    }
  }

  return data;
}

function resolveScopedSlots(
  fns,
  res,

  // the following are added in 2.6

  hasDynamicKeys,
  contentHashKey
) {
  res = res || { $stable: !hasDynamicKeys };

  for (var i = 0; i < fns.length; i++) {
    var slot = fns[i];

    if (isArray(slot)) {
      resolveScopedSlots(slot, res, hasDynamicKeys);
    } else if (slot) {
      // marker for reverse proxying v-slot without scope on this.$slots

      // @ts-expect-error

      if (slot.proxy) {
        // @ts-expect-error

        slot.fn.proxy = true;
      }

      res[slot.key] = slot.fn;
    }
  }

  if (contentHashKey) {
    res.$key = contentHashKey;
  }

  return res;
}

// helper to process dynamic keys for dynamic arguments in v-bind and v-on.

function bindDynamicKeys(baseObj, values) {
  for (var i = 0; i < values.length; i += 2) {
    var key = values[i];

    if (typeof key === "string" && key) {
      baseObj[values[i]] = values[i + 1];
    } else if (
      process.env.NODE_ENV !== "production" &&
      key !== "" &&
      key !== null
    ) {
      // null is a special value for explicitly removing a binding

      warn$2(
        "Invalid value for dynamic directive argument (expected string or null): ".concat(
          key
        ),
        this
      );
    }
  }

  return baseObj;
}

// helper to dynamically append modifier runtime markers to event names.

// ensure only append when value is already string, otherwise it will be cast

// to string and cause the type check to miss.

function prependModifier(value, symbol) {
  return typeof value === "string" ? symbol + value : value;
}

function installRenderHelpers(target) {
  target._o = markOnce;

  target._n = toNumber;

  target._s = toString;

  target._l = renderList;

  target._t = renderSlot;

  target._q = looseEqual;

  target._i = looseIndexOf;

  target._m = renderStatic;

  target._f = resolveFilter;

  target._k = checkKeyCodes;

  target._b = bindObjectProps;

  target._v = createTextVNode;

  target._e = createEmptyVNode;

  target._u = resolveScopedSlots;

  target._g = bindObjectListeners;

  target._d = bindDynamicKeys;

  target._p = prependModifier;
}

/**

 * Runtime helper for resolving raw children VNodes into a slot object.

 */

function resolveSlots(children, context) {
  if (!children || !children.length) {
    return {};
  }

  var slots = {};

  for (var i = 0, l = children.length; i < l; i++) {
    var child = children[i];

    var data = child.data; // remove slot attribute if the node is resolved as a Vue slot node

    if (data && data.attrs && data.attrs.slot) {
      delete data.attrs.slot;
    } // named slots should only be respected if the vnode was rendered in the // same context.

    if (
      (child.context === context || child.fnContext === context) &&
      data &&
      data.slot != null
    ) {
      var name_1 = data.slot;

      var slot = slots[name_1] || (slots[name_1] = []);

      if (child.tag === "template") {
        slot.push.apply(slot, child.children || []);
      } else {
        slot.push(child);
      }
    } else {
      (slots.default || (slots.default = [])).push(child);
    }
  } // ignore slots that contains only whitespace

  for (var name_2 in slots) {
    if (slots[name_2].every(isWhitespace)) {
      delete slots[name_2];
    }
  }

  return slots;
}

function isWhitespace(node) {
  return (node.isComment && !node.asyncFactory) || node.text === " ";
}

function isAsyncPlaceholder(node) {
  // @ts-expect-error not really boolean type

  return node.isComment && node.asyncFactory;
}
```

## 2209~2245 解释

这段代码定义了一些 Vue.js 运行时的帮助函数，包括解析过滤器、检查按键码、绑定对象属性和监听器、渲染静态节点等功能。其中 `resolveFilter` 函数用于解析过滤器，`checkKeyCodes` 函数用于检查按键码，`bindObjectProps` 函数用于绑定对象属性，`bindObjectListeners` 函数用于绑定对象监听器，`renderStatic` 函数用于渲染静态节点，`resolveSlots` 函数用于解析插槽内容等。这些函数都是 Vue.js 运行时的核心帮助函数，可以提高组件的性能和效率。

# 2246_2451

```js
function normalizeScopedSlots(
  ownerVm,
  scopedSlots,
  normalSlots,
  prevScopedSlots
) {
  var res;

  var hasNormalSlots = Object.keys(normalSlots).length > 0;

  var isStable = scopedSlots ? !!scopedSlots.$stable : !hasNormalSlots;

  var key = scopedSlots && scopedSlots.$key;

  if (!scopedSlots) {
    res = {};
  } else if (scopedSlots._normalized) {
    // fast path 1: child component re-render only, parent did not change

    return scopedSlots._normalized;
  } else if (
    isStable &&
    prevScopedSlots &&
    prevScopedSlots !== emptyObject &&
    key === prevScopedSlots.$key &&
    !hasNormalSlots &&
    !prevScopedSlots.$hasNormal
  ) {
    // fast path 2: stable scoped slots w/ no normal slots to proxy,

    // only need to normalize once

    return prevScopedSlots;
  } else {
    res = {};

    for (var key_1 in scopedSlots) {
      if (scopedSlots[key_1] && key_1[0] !== "$") {
        res[key_1] = normalizeScopedSlot(
          ownerVm,
          normalSlots,
          key_1,
          scopedSlots[key_1]
        );
      }
    }
  } // expose normal slots on scopedSlots

  for (var key_2 in normalSlots) {
    if (!(key_2 in res)) {
      res[key_2] = proxyNormalSlot(normalSlots, key_2);
    }
  } // avoriaz seems to mock a non-extensible $scopedSlots object // and when that is passed down this would cause an error

  if (scopedSlots && Object.isExtensible(scopedSlots)) {
    scopedSlots._normalized = res;
  }

  def(res, "$stable", isStable);

  def(res, "$key", key);

  def(res, "$hasNormal", hasNormalSlots);

  return res;
}

function normalizeScopedSlot(vm, normalSlots, key, fn) {
  var normalized = function () {
    var cur = currentInstance;

    setCurrentInstance(vm);

    var res = arguments.length ? fn.apply(null, arguments) : fn({});

    res =
      res && typeof res === "object" && !isArray(res)
        ? [res] // single vnode
        : normalizeChildren(res);

    var vnode = res && res[0];

    setCurrentInstance(cur);

    return res &&
      (!vnode ||
        (res.length === 1 && vnode.isComment && !isAsyncPlaceholder(vnode))) // #9658, #10391
      ? undefined
      : res;
  }; // this is a slot using the new v-slot syntax without scope. although it is // compiled as a scoped slot, render fn users would expect it to be present // on this.$slots because the usage is semantically a normal slot.

  if (fn.proxy) {
    Object.defineProperty(normalSlots, key, {
      get: normalized,

      enumerable: true,

      configurable: true,
    });
  }

  return normalized;
}

function proxyNormalSlot(slots, key) {
  return function () {
    return slots[key];
  };
}

function initSetup(vm) {
  var options = vm.$options;

  var setup = options.setup;

  if (setup) {
    var ctx = (vm._setupContext = createSetupContext(vm));

    setCurrentInstance(vm);

    pushTarget();

    var setupResult = invokeWithErrorHandling(
      setup,
      null,
      [vm._props || shallowReactive({}), ctx],
      vm,
      "setup"
    );

    popTarget();

    setCurrentInstance();

    if (isFunction(setupResult)) {
      // render function

      // @ts-ignore

      options.render = setupResult;
    } else if (isObject(setupResult)) {
      // bindings

      if (
        process.env.NODE_ENV !== "production" &&
        setupResult instanceof VNode
      ) {
        warn$2(
          "setup() should not return VNodes directly - " +
            "return a render function instead."
        );
      }

      vm._setupState = setupResult; // __sfc indicates compiled bindings from <script setup>

      if (!setupResult.__sfc) {
        for (var key in setupResult) {
          if (!isReserved(key)) {
            proxyWithRefUnwrap(vm, setupResult, key);
          } else if (process.env.NODE_ENV !== "production") {
            warn$2("Avoid using variables that start with _ or $ in setup().");
          }
        }
      } else {
        // exposed for compiled render fn

        var proxy = (vm._setupProxy = {});

        for (var key in setupResult) {
          if (key !== "__sfc") {
            proxyWithRefUnwrap(proxy, setupResult, key);
          }
        }
      }
    } else if (
      process.env.NODE_ENV !== "production" &&
      setupResult !== undefined
    ) {
      warn$2(
        "setup() should return an object. Received: ".concat(
          setupResult === null ? "null" : typeof setupResult
        )
      );
    }
  }
}

function createSetupContext(vm) {
  var exposeCalled = false;

  return {
    get attrs() {
      if (!vm._attrsProxy) {
        var proxy = (vm._attrsProxy = {});

        def(proxy, "_v_attr_proxy", true);

        syncSetupProxy(proxy, vm.$attrs, emptyObject, vm, "$attrs");
      }

      return vm._attrsProxy;
    },

    get listeners() {
      if (!vm._listenersProxy) {
        var proxy = (vm._listenersProxy = {});

        syncSetupProxy(proxy, vm.$listeners, emptyObject, vm, "$listeners");
      }

      return vm._listenersProxy;
    },

    get slots() {
      return initSlotsProxy(vm);
    },

    emit: bind$1(vm.$emit, vm),

    expose: function (exposed) {
      if (process.env.NODE_ENV !== "production") {
        if (exposeCalled) {
          warn$2("expose() should be called only once per setup().", vm);
        }

        exposeCalled = true;
      }

      if (exposed) {
        Object.keys(exposed).forEach(function (key) {
          return proxyWithRefUnwrap(vm, exposed, key);
        });
      }
    },
  };
}

function syncSetupProxy(to, from, prev, instance, type) {
  var changed = false;

  for (var key in from) {
    if (!(key in to)) {
      changed = true;

      defineProxyAttr(to, key, instance, type);
    } else if (from[key] !== prev[key]) {
      changed = true;
    }
  }

  for (var key in to) {
    if (!(key in from)) {
      changed = true;

      delete to[key];
    }
  }

  return changed;
}

function defineProxyAttr(proxy, key, instance, type) {
  Object.defineProperty(proxy, key, {
    enumerable: true,

    configurable: true,

    get: function () {
      return instance[type][key];
    },
  });
}

function initSlotsProxy(vm) {
  if (!vm._slotsProxy) {
    syncSetupSlots((vm._slotsProxy = {}), vm.$scopedSlots);
  }

  return vm._slotsProxy;
}

function syncSetupSlots(to, from) {
  for (var key in from) {
    to[key] = from[key];
  }

  for (var key in to) {
    if (!(key in from)) {
      delete to[key];
    }
  }
}

/**

 * @internal use manual type def because public setup context type relies on

 * legacy VNode types

 */

function useSlots() {
  return getContext().slots;
}

/**

 * @internal use manual type def because public setup context type relies on

 * legacy VNode types

 */

function useAttrs() {
  return getContext().attrs;
}

/**

 * Vue 2 only

 * @internal use manual type def because public setup context type relies on

 * legacy VNode types

 */

function useListeners() {
  return getContext().listeners;
}

function getContext() {
  if (process.env.NODE_ENV !== "production" && !currentInstance) {
    warn$2("useContext() called without active instance.");
  }

  var vm = currentInstance;

  return vm._setupContext || (vm._setupContext = createSetupContext(vm));
}
```

## 2246_2451 解释

这段代码是 Vue.js 的运行时核心代码，包括帮助函数和初始化设置相关的函数。其中，normalizeScopedSlots 函数用于规范化作用域插槽，initSetup 函数用于初始化组件的 setup 函数，createSetupContext 函数用于创建 setup 上下文，syncSetupProxy 函数用于同步 setup 代理属性，defineProxyAttr 函数用于定义代理属性，initSlotsProxy 函数用于初始化 slots 代理，useSlots、useAttrs 和 useListeners 函数用于获取当前的 slots、attrs 和 listeners 对象，getContext 函数用于获取当前的 setup 上下文。这些函数为 Vue.js 提供了强大的能力，使得开发者可以更加方便地编写高效、灵活、可维护的组件。

# 2452~2707

```js
unction mergeDefaults(raw, defaults) {

    var props = isArray(raw)

        ? raw.reduce(function (normalized, p) { return ((normalized[p] = {}), normalized); }, {})

        : raw;

    for (var key in defaults) {

        var opt = props[key];

        if (opt) {

            if (isArray(opt) || isFunction(opt)) {

                props[key] = { type: opt, default: defaults[key] };

            }

            else {

                opt.default = defaults[key];

            }

        }

        else if (opt === null) {

            props[key] = { default: defaults[key] };

        }

        else if (process.env.NODE_ENV !== 'production') {

            warn$2("props default key \"".concat(key, "\" has no corresponding declaration."));

        }

    }

    return props;

}



function initRender(vm) {

    vm._vnode = null; // the root of the child tree

    vm._staticTrees = null; // v-once cached trees

    var options = vm.$options;

    var parentVnode = (vm.$vnode = options._parentVnode); // the placeholder node in parent tree

    var renderContext = parentVnode && parentVnode.context;

    vm.$slots = resolveSlots(options._renderChildren, renderContext);

    vm.$scopedSlots = parentVnode

        ? normalizeScopedSlots(vm.$parent, parentVnode.data.scopedSlots, vm.$slots)

        : emptyObject;

    // bind the createElement fn to this instance

    // so that we get proper render context inside it.

    // args order: tag, data, children, normalizationType, alwaysNormalize

    // internal version is used by render functions compiled from templates

    // @ts-expect-error

    vm._c = function (a, b, c, d) { return createElement$1(vm, a, b, c, d, false); };

    // normalization is always applied for the public version, used in

    // user-written render functions.

    // @ts-expect-error

    vm.$createElement = function (a, b, c, d) { return createElement$1(vm, a, b, c, d, true); };

    // $attrs & $listeners are exposed for easier HOC creation.

    // they need to be reactive so that HOCs using them are always updated

    var parentData = parentVnode && parentVnode.data;

    /* istanbul ignore else */

    if (process.env.NODE_ENV !== 'production') {

        defineReactive(vm, '$attrs', (parentData && parentData.attrs) || emptyObject, function () {

            !isUpdatingChildComponent && warn$2("$attrs is readonly.", vm);

        }, true);

        defineReactive(vm, '$listeners', options._parentListeners || emptyObject, function () {

            !isUpdatingChildComponent && warn$2("$listeners is readonly.", vm);

        }, true);

    }

    else {

        defineReactive(vm, '$attrs', (parentData && parentData.attrs) || emptyObject, null, true);

        defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true);

    }

}

var currentRenderingInstance = null;

function renderMixin(Vue) {

    // install runtime convenience helpers

    installRenderHelpers(Vue.prototype);

    Vue.prototype.$nextTick = function (fn) {

        return nextTick(fn, this);

    };

    Vue.prototype._render = function () {

        var vm = this;

        var _a = vm.$options, render = _a.render, _parentVnode = _a._parentVnode;

        if (_parentVnode && vm._isMounted) {

            vm.$scopedSlots = normalizeScopedSlots(vm.$parent, _parentVnode.data.scopedSlots, vm.$slots, vm.$scopedSlots);

            if (vm._slotsProxy) {

                syncSetupSlots(vm._slotsProxy, vm.$scopedSlots);

            }

        }

        // set parent vnode. this allows render functions to have access

        // to the data on the placeholder node.

        vm.$vnode = _parentVnode;

        // render self

        var vnode;

        try {

            // There's no need to maintain a stack because all render fns are called

            // separately from one another. Nested component's render fns are called

            // when parent component is patched.

            setCurrentInstance(vm);

            currentRenderingInstance = vm;

            vnode = render.call(vm._renderProxy, vm.$createElement);

        }

        catch (e) {

            handleError(e, vm, "render");

            // return error render result,

            // or previous vnode to prevent render error causing blank component

            /* istanbul ignore else */

            if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {

                try {

                    vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e);

                }

                catch (e) {

                    handleError(e, vm, "renderError");

                    vnode = vm._vnode;

                }

            }

            else {

                vnode = vm._vnode;

            }

        }

        finally {

            currentRenderingInstance = null;

            setCurrentInstance();

        }

        // if the returned array contains only a single node, allow it

        if (isArray(vnode) && vnode.length === 1) {

            vnode = vnode[0];

        }

        // return empty vnode in case the render function errored out

        if (!(vnode instanceof VNode)) {

            if (process.env.NODE_ENV !== 'production' && isArray(vnode)) {

                warn$2('Multiple root nodes returned from render function. Render function ' +

                    'should return a single root node.', vm);

            }

            vnode = createEmptyVNode();

        }

        // set parent

        vnode.parent = _parentVnode;

        return vnode;

    };

}



function ensureCtor(comp, base) {

    if (comp.__esModule || (hasSymbol && comp[Symbol.toStringTag] === 'Module')) {

        comp = comp.default;

    }

    return isObject(comp) ? base.extend(comp) : comp;

}

function createAsyncPlaceholder(factory, data, context, children, tag) {

    var node = createEmptyVNode();

    node.asyncFactory = factory;

    node.asyncMeta = { data: data, context: context, children: children, tag: tag };

    return node;

}

function resolveAsyncComponent(factory, baseCtor) {

    if (isTrue(factory.error) && isDef(factory.errorComp)) {

        return factory.errorComp;

    }

    if (isDef(factory.resolved)) {

        return factory.resolved;

    }

    var owner = currentRenderingInstance;

    if (owner && isDef(factory.owners) && factory.owners.indexOf(owner) === -1) {

        // already pending

        factory.owners.push(owner);

    }

    if (isTrue(factory.loading) && isDef(factory.loadingComp)) {

        return factory.loadingComp;

    }

    if (owner && !isDef(factory.owners)) {

        var owners_1 = (factory.owners = [owner]);

        var sync_1 = true;

        var timerLoading_1 = null;

        var timerTimeout_1 = null;

        owner.$on('hook:destroyed', function () { return remove$2(owners_1, owner); });

        var forceRender_1 = function (renderCompleted) {

            for (var i = 0, l = owners_1.length; i < l; i++) {

                owners_1[i].$forceUpdate();

            }

            if (renderCompleted) {

                owners_1.length = 0;

                if (timerLoading_1 !== null) {

                    clearTimeout(timerLoading_1);

                    timerLoading_1 = null;

                }

                if (timerTimeout_1 !== null) {

                    clearTimeout(timerTimeout_1);

                    timerTimeout_1 = null;

                }

            }

        };

        var resolve = once(function (res) {

            // cache resolved

            factory.resolved = ensureCtor(res, baseCtor);

            // invoke callbacks only if this is not a synchronous resolve

            // (async resolves are shimmed as synchronous during SSR)

            if (!sync_1) {

                forceRender_1(true);

            }

            else {

                owners_1.length = 0;

            }

        });

        var reject_1 = once(function (reason) {

            process.env.NODE_ENV !== 'production' &&

                warn$2("Failed to resolve async component: ".concat(String(factory)) +

                    (reason ? "\nReason: ".concat(reason) : ''));

            if (isDef(factory.errorComp)) {

                factory.error = true;

                forceRender_1(true);

            }

        });

        var res_1 = factory(resolve, reject_1);

        if (isObject(res_1)) {

            if (isPromise(res_1)) {

                // () => Promise

                if (isUndef(factory.resolved)) {

                    res_1.then(resolve, reject_1);

                }

            }

            else if (isPromise(res_1.component)) {

                res_1.component.then(resolve, reject_1);

                if (isDef(res_1.error)) {

                    factory.errorComp = ensureCtor(res_1.error, baseCtor);

                }

                if (isDef(res_1.loading)) {

                    factory.loadingComp = ensureCtor(res_1.loading, baseCtor);

                    if (res_1.delay === 0) {

                        factory.loading = true;

                    }

                    else {

                        // @ts-expect-error NodeJS timeout type

                        timerLoading_1 = setTimeout(function () {

                            timerLoading_1 = null;

                            if (isUndef(factory.resolved) && isUndef(factory.error)) {

                                factory.loading = true;

                                forceRender_1(false);

                            }

                        }, res_1.delay || 200);

                    }

                }

                if (isDef(res_1.timeout)) {

                    // @ts-expect-error NodeJS timeout type

                    timerTimeout_1 = setTimeout(function () {

                        timerTimeout_1 = null;

                        if (isUndef(factory.resolved)) {

                            reject_1(process.env.NODE_ENV !== 'production' ? "timeout (".concat(res_1.timeout, "ms)") : null);

                        }

                    }, res_1.timeout);

                }

            }

        }

        sync_1 = false;

        // return in case resolved synchronously

        return factory.loading ? factory.loadingComp : factory.resolved;

    }

}



function getFirstComponentChild(children) {

    if (isArray(children)) {

        for (var i = 0; i < children.length; i++) {

            var c = children[i];

            if (isDef(c) && (isDef(c.componentOptions) || isAsyncPlaceholder(c))) {

                return c;

            }

        }

    }

}
```

## 2452~2707 解释

这段代码是 Vue.js 的渲染相关的 mixin，定义了一些与组件渲染相关的函数和逻辑。其中包括 mergeDefaults 函数用于合并默认 prop 值，initRender 函数用于初始化渲染环境，renderMixin 函数用于安装渲染相关的实例方法和定义 \_render 函数用于组件渲染。另外还包括一些辅助函数，如 ensureCtor 用于转化异步组件为构造器，createAsyncPlaceholder 用于创建异步组件的占位符节点，resolveAsyncComponent 用于解析异步组件，getFirstComponentChild 用于获取子节点中的第一个组件节点。这些函数为 Vue.js 提供了强大的渲染能力，可以使开发者更加方便地编写高效、灵活、可维护的组件。

# 2709~3013

```js
function initEvents(vm) {
  vm._events = Object.create(null);

  vm._hasHookEvent = false; // init parent attached events

  var listeners = vm.$options._parentListeners;

  if (listeners) {
    updateComponentListeners(vm, listeners);
  }
}

var target$1;

function add$1(event, fn) {
  target$1.$on(event, fn);
}

function remove$1(event, fn) {
  target$1.$off(event, fn);
}

function createOnceHandler$1(event, fn) {
  var _target = target$1;

  return function onceHandler() {
    var res = fn.apply(null, arguments);

    if (res !== null) {
      _target.$off(event, onceHandler);
    }
  };
}

function updateComponentListeners(vm, listeners, oldListeners) {
  target$1 = vm;

  updateListeners(
    listeners,
    oldListeners || {},
    add$1,
    remove$1,
    createOnceHandler$1,
    vm
  );

  target$1 = undefined;
}

function eventsMixin(Vue) {
  var hookRE = /^hook:/;

  Vue.prototype.$on = function (event, fn) {
    var vm = this;

    if (isArray(event)) {
      for (var i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn);
      }
    } else {
      (vm._events[event] || (vm._events[event] = [])).push(fn); // optimize hook:event cost by using a boolean flag marked at registration // instead of a hash lookup

      if (hookRE.test(event)) {
        vm._hasHookEvent = true;
      }
    }

    return vm;
  };

  Vue.prototype.$once = function (event, fn) {
    var vm = this;

    function on() {
      vm.$off(event, on);

      fn.apply(vm, arguments);
    }

    on.fn = fn;

    vm.$on(event, on);

    return vm;
  };

  Vue.prototype.$off = function (event, fn) {
    var vm = this; // all

    if (!arguments.length) {
      vm._events = Object.create(null);

      return vm;
    } // array of events

    if (isArray(event)) {
      for (var i_1 = 0, l = event.length; i_1 < l; i_1++) {
        vm.$off(event[i_1], fn);
      }

      return vm;
    } // specific event

    var cbs = vm._events[event];

    if (!cbs) {
      return vm;
    }

    if (!fn) {
      vm._events[event] = null;

      return vm;
    } // specific handler

    var cb;

    var i = cbs.length;

    while (i--) {
      cb = cbs[i];

      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1);

        break;
      }
    }

    return vm;
  };

  Vue.prototype.$emit = function (event) {
    var vm = this;

    if (process.env.NODE_ENV !== "production") {
      var lowerCaseEvent = event.toLowerCase();

      if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
        tip(
          'Event "'.concat(lowerCaseEvent, '" is emitted in component ') +
            ""
              .concat(
                formatComponentName(vm),
                ' but the handler is registered for "'
              )
              .concat(event, '". ') +
            "Note that HTML attributes are case-insensitive and you cannot use " +
            "v-on to listen to camelCase events when using in-DOM templates. " +
            'You should probably use "'
              .concat(hyphenate(event), '" instead of "')
              .concat(event, '".')
        );
      }
    }

    var cbs = vm._events[event];

    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs;

      var args = toArray(arguments, 1);

      var info = 'event handler for "'.concat(event, '"');

      for (var i = 0, l = cbs.length; i < l; i++) {
        invokeWithErrorHandling(cbs[i], vm, args, vm, info);
      }
    }

    return vm;
  };
}

var activeInstance = null;

var isUpdatingChildComponent = false;

function setActiveInstance(vm) {
  var prevActiveInstance = activeInstance;

  activeInstance = vm;

  return function () {
    activeInstance = prevActiveInstance;
  };
}

function initLifecycle(vm) {
  var options = vm.$options; // locate first non-abstract parent

  var parent = options.parent;

  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent;
    }

    parent.$children.push(vm);
  }

  vm.$parent = parent;

  vm.$root = parent ? parent.$root : vm;

  vm.$children = [];

  vm.$refs = {};

  vm._provided = parent ? parent._provided : Object.create(null);

  vm._watcher = null;

  vm._inactive = null;

  vm._directInactive = false;

  vm._isMounted = false;

  vm._isDestroyed = false;

  vm._isBeingDestroyed = false;
}

function lifecycleMixin(Vue) {
  Vue.prototype._update = function (vnode, hydrating) {
    var vm = this;

    var prevEl = vm.$el;

    var prevVnode = vm._vnode;

    var restoreActiveInstance = setActiveInstance(vm);

    vm._vnode = vnode; // Vue.prototype.__patch__ is injected in entry points // based on the rendering backend used.

    if (!prevVnode) {
      // initial render

      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */);
    } else {
      // updates

      vm.$el = vm.__patch__(prevVnode, vnode);
    }

    restoreActiveInstance(); // update __vue__ reference

    if (prevEl) {
      prevEl.__vue__ = null;
    }

    if (vm.$el) {
      vm.$el.__vue__ = vm;
    } // if parent is an HOC, update its $el as well

    var wrapper = vm;

    while (
      wrapper &&
      wrapper.$vnode &&
      wrapper.$parent &&
      wrapper.$vnode === wrapper.$parent._vnode
    ) {
      wrapper.$parent.$el = wrapper.$el;

      wrapper = wrapper.$parent;
    } // updated hook is called by the scheduler to ensure that children are // updated in a parent's updated hook.
  };

  Vue.prototype.$forceUpdate = function () {
    var vm = this;

    if (vm._watcher) {
      vm._watcher.update();
    }
  };

  Vue.prototype.$destroy = function () {
    var vm = this;

    if (vm._isBeingDestroyed) {
      return;
    }

    callHook$1(vm, "beforeDestroy");

    vm._isBeingDestroyed = true; // remove self from parent

    var parent = vm.$parent;

    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove$2(parent.$children, vm);
    } // teardown scope. this includes both the render watcher and other // watchers created

    vm._scope.stop(); // remove reference from data ob // frozen object may not have observer.

    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--;
    } // call the last hook...

    vm._isDestroyed = true; // invoke destroy hooks on current rendered tree

    vm.__patch__(vm._vnode, null); // fire destroyed hook

    callHook$1(vm, "destroyed"); // turn off all instance listeners.

    vm.$off(); // remove __vue__ reference

    if (vm.$el) {
      vm.$el.__vue__ = null;
    } // release circular reference (#6759)

    if (vm.$vnode) {
      vm.$vnode.parent = null;
    }
  };
}

function mountComponent(vm, el, hydrating) {
  vm.$el = el;

  if (!vm.$options.render) {
    // @ts-expect-error invalid type

    vm.$options.render = createEmptyVNode;

    if (process.env.NODE_ENV !== "production") {
      /* istanbul ignore if */

      if (
        (vm.$options.template && vm.$options.template.charAt(0) !== "#") ||
        vm.$options.el ||
        el
      ) {
        warn$2(
          "You are using the runtime-only build of Vue where the template " +
            "compiler is not available. Either pre-compile the templates into " +
            "render functions, or use the compiler-included build.",
          vm
        );
      } else {
        warn$2(
          "Failed to mount component: template or render function not defined.",
          vm
        );
      }
    }
  }

  callHook$1(vm, "beforeMount");

  var updateComponent; /* istanbul ignore if */

  if (process.env.NODE_ENV !== "production" && config.performance && mark) {
    updateComponent = function () {
      var name = vm._name;

      var id = vm._uid;

      var startTag = "vue-perf-start:".concat(id);

      var endTag = "vue-perf-end:".concat(id);

      mark(startTag);

      var vnode = vm._render();

      mark(endTag);

      measure("vue ".concat(name, " render"), startTag, endTag);

      mark(startTag);

      vm._update(vnode, hydrating);

      mark(endTag);

      measure("vue ".concat(name, " patch"), startTag, endTag);
    };
  } else {
    updateComponent = function () {
      vm._update(vm._render(), hydrating);
    };
  }

  var watcherOptions = {
    before: function () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook$1(vm, "beforeUpdate");
      }
    },
  };

  if (process.env.NODE_ENV !== "production") {
    watcherOptions.onTrack = function (e) {
      return callHook$1(vm, "renderTracked", [e]);
    };

    watcherOptions.onTrigger = function (e) {
      return callHook$1(vm, "renderTriggered", [e]);
    };
  } // we set this to vm._watcher inside the watcher's constructor // since the watcher's initial patch may call $forceUpdate (e.g. inside child // component's mounted hook), which relies on vm._watcher being already defined

  new Watcher(
    vm,
    updateComponent,
    noop,
    watcherOptions,
    true /* isRenderWatcher */
  );

  hydrating = false; // flush buffer for flush: "pre" watchers queued in setup()

  var preWatchers = vm._preWatchers;

  if (preWatchers) {
    for (var i = 0; i < preWatchers.length; i++) {
      preWatchers[i].run();
    }
  } // manually mounted instance, call mounted on self // mounted is called for render-created child components in its inserted hook

  if (vm.$vnode == null) {
    vm._isMounted = true;

    callHook$1(vm, "mounted");
  }

  return vm;
}
```

## 2709~3013 解释

这段代码主要是 Vue.js 框架中的事件处理和生命周期管理相关的函数。其中，eventsMixin 函数用于给 Vue 实例添加事件处理相关的方法，如 on、off 和 emit 等；lifecycleMixin 函数用于给 Vue 实例添加生命周期处理相关的方法，如 u​pdate、forceUpdate 和 $destroy 等。除此之外，还有一些辅助函数，如 setActiveInstance 函数用于设置当前活跃的 Vue 实例，initEvents 函数用于初始化事件相关的参数，initLifecycle 函数用于初始化生命周期相关的参数，mountComponent 函数用于挂载组件等。这些函数共同构成了 Vue.js 框架的核心部分，能够帮助开发者更方便地管理组件的事件和生命周期。
