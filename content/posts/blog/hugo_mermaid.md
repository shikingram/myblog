{
  "title":"不使用shortcode渲染mermaid",
  "tags":[
    "blog"
  ],
  "date":"2021-08-21",
  "lastmod":"2021-08-21",
  "draft":"false",
  "author":"kingram"
}

ShortCode语法和很多前端语法冲突，甚至和我的golang Template语法冲突。

我想在我本地的typora中能打开，也能在网页中打开，终于找到了一个可以用的方案，而且效果还好。

## 在Hugo中配置mermaid
在themes/主题名/layouts/partials目录下修改footer.html文件增加以下内容：
```
<script src="https://unpkg.com/mermaid@8.8.1/dist/mermaid.min.js"></script>
  <script>
        Array.from(document.getElementsByClassName('language-mermaid')).forEach(el => {
        el.parentElement.outerHTML = `<div class="mermaid">${el.innerText}</div>`
    })
</script>
```
## 测试用例
测试例子全部来自：[mermaid-js](https://mermaid-js.github.io/mermaid/#/)
```mermaid
graph LR
    id1[(Database)]
```


```mermaid
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John-->>Alice: Great!
    Alice-)John: See you later!
```

```mermaid
pie
    title Key elements in Product X
    "Calcium" : 42.96
    "Potassium" : 50.05
    "Magnesium" : 10.01
    "Iron" :  5
```


```mermaid
    requirementDiagram

    requirement test_req {
    id: 1
    text: the test text.
    risk: high
    verifymethod: test
    }

    element test_entity {
    type: simulation
    }

    test_entity - satisfies -> test_req
```


```mermaid
journey
    title My working day
    section Go to work
      Make tea: 5: Me
      Go upstairs: 3: Me
      Do work: 1: Me, Cat
    section Go home
      Go downstairs: 5: Me
      Sit down: 5: Me
```


```mermaid
gitGraph:
options
{
    "nodeSpacing": 150,
    "nodeRadius": 10
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
merge newbranch
```
