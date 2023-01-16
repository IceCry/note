### 

> ```
> 😑 日常笔记
> ```

<div class="demo-theme-preview">
  <a data-theme="vue">白昼</a>
  <a data-theme="dark">暗夜</a>
</div>

<style>
  .demo-theme-preview a {
    padding-right: 10px;
  }

  .demo-theme-preview a:hover {
    cursor: pointer;
    text-decoration: underline;
  }
</style>

<script>
  var preview = Docsify.dom.find('.demo-theme-preview');
  var themes = Docsify.dom.findAll('[rel="stylesheet"]');


  preview.onclick = function (e) {
    var title = e.target.getAttribute('data-theme');
  	console.log(title)

    themes.forEach(function (theme) {
      theme.disabled = theme.title !== title;
    });
  };
</script>
