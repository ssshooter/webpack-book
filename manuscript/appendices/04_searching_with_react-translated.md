＃使用React搜索
# Searching with React

假设你想要在没有适当后端的情况下对应用程序进行粗略的小搜索。你可以通过[lunr]（http://lunrjs.com/）完成此操作并生成静态搜索索引。
Let's say you want to implement a rough little search for an application without a proper backend. You could do it through [lunr](http://lunrjs.com/) and generate a static search index to serve.

问题是索引可能很大，具体取决于内容的数量。好处是你从一开始就不需要搜索索引。你可以做一些更聪明的事情。你可以在用户选择搜索字段时开始加载索引。
The problem is that the index can be sizable depending on the amount of the content. The good thing is that you don't need the search index straight from the start. You can do something smarter instead. You can start loading the index when the user selects a search field.

这样做会延迟加载并将其移动到性能更可接受的地方。初始搜索将比后续搜索慢，你应该显示加载指示符。但从用户的角度来看，这很好。 Webpack的* Code Splitting *功能允许这样做。
Doing this defers the loading and moves it to a place where it's more acceptable for performance. The initial search is going to be slower than the subsequent ones, and you should display a loading indicator. But that's fine from the user point of view. Webpack's *Code Splitting* feature allows doing this.

##使用代码拆分实现搜索
## Implementing Search with Code Splitting

要实现代码拆分，你需要决定将拆分点放在哪里，将其放在那里，然后处理`Promise`：
To implement code splitting, you need to decide where to put the split point, put it there, and then handle the `Promise`:

```javascript
import("./asset").then(asset => ...).catch(err => ...)
```

美妙的是，这可以在出现问题（网络故障等）时提供错误处理，并提供恢复的机会。你还可以使用基于“Promise”的实用程序（如“Promise.all”）来编写更复杂的查询。
The beautiful thing is that this gives error handling in case something goes wrong (network is down etc.) and gives a chance to recover. You can also use `Promise` based utilities like `Promise.all` for composing more complicated queries.

{pagebreak}

在这种情况下，你需要检测用户何时选择搜索元素，加载数据，除非已经加载了数据，然后对其执行搜索逻辑。考虑下面的React实现：
In this case, you need to detect when the user selects the search element, load the data unless it has been loaded already, and then execute search logic against it. Consider the React implementation below:

** ** App.js
**App.js**

```javascript
import React from "react";

export default class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      index: null,
      value: "",
      lines: [],
      results: [],
    };
  }
  render() {
    const { results, value } = this.state;

    return (
      <div className="app-container">
        <div className="search-container">
          <label>Search against README:</label>
          <input
            type="text"
            value={value}
            onChange={e => this.onChange(e)}
          />
        </div>
        <div className="results-container">
          <Results results={results} />
        </div>
      </div>
    );
  }
  onChange({ target: { value } }) {
    const { index, lines } = this.state;

    // Set captured value to input
    this.setState(() => ({ value }));

    // Search against lines and index if they exist
    if (lines && index) {
      return this.setState(() => ({
        results: this.search(lines, index, value),
      }));
    }

    // If the index doesn't exist, it has to be set it up.
    // You could show loading indicator here as loading might
    // take a while depending on the size of the index.
    loadIndex()
      .then(({ index, lines }) => {
        // Search against the index now.
        this.setState(() => ({
          index,
          lines,
          results: this.search(lines, index, value),
        }));
      })
      .catch(err => console.error(err));
  }
  search(lines, index, query) {
    // Search against the index and match README lines.
    return index
      .search(query.trim())
      .map(match => lines[match.ref]);
  }
}

const Results = ({ results }) => {
  if (results.length) {
    return (
      <ul>
        {results.map((result, i) => <li key={i}>{result}</li>)}
      </ul>
    );
  }

  return <span>No results</span>;
};

function loadIndex() {
  // Here's the magic. Set up `import` to tell Webpack
  // to split here and load our search index dynamically.
  //
  // Note that you will need to shim Promise.all for
  // older browsers and Internet Explorer!
  return Promise.all([
    import("lunr"),
    import("../search_index.json"),
  ]).then(([{ Index }, { index, lines }]) => {
    return {
      index: Index.load(index),
      lines,
    };
  });
}
```

在示例中，webpack静态检测到`import`。它可以基于此分割点生成单独的捆绑包。鉴于它依赖于静态分析，在这种情况下你不能概括`loadIndex`并将搜索索引路径作为参数传递。
In the example, webpack detects the `import` statically. It can generate a separate bundle based on this split point. Given it relies on static analysis, you cannot generalize `loadIndex` in this case and pass the search index path as a parameter.

{pagebreak}

##结论
## Conclusion

除了搜索之外，该方法也可以与路由器一起使用。当用户输入路由时，你可以加载生成的视图所需的依赖项。或者，你可以在用户滚动页面并使用实际功能获取相邻部件时开始加载依赖项。 `import`提供了很多功能，可以让你的应用程序保持精简状态。
Beyond search, the approach can be used with routers too. As the user enters a route, you can load the dependencies the resulting view needs. Alternately, you can start loading dependencies as the user scrolls a page and gets adjacent parts with actual functionality. `import` provides a lot of power and allows you to keep your application lean.

你可以找到一个[完整的例子]（https://github.com/survivejs-demos/lunr-demo），展示它与lunr，React和webpack的结合。基本想法是一样的，但有更多的设置。
You can find a [full example](https://github.com/survivejs-demos/lunr-demo) showing how it all goes together with lunr, React, and webpack. The basic idea is the same, but there's more setup in place.

回顾一下：
To recap:

*如果你的数据集很小且是静态的，那么客户端搜索是一个不错的选择。
* If your dataset is small and static, client-side search is a good option.
*你可以使用像[lunr]（http://lunrjs.com/）这样的解决方案索引你的内容，然后对其进行搜索。
* You can index your content using a solution like [lunr](http://lunrjs.com/) and then perform a search against it.
* Webpack的*代码分割*功能非常适合按需加载搜索索引。
* Webpack's *code splitting* feature is ideal for loading a search index on demand.
*代码拆分可以与React等UI解决方案结合使用，以实现整个用户界面。
* Code splitting can be combined with a UI solution like React to implement the whole user interface.

