path: path/to/file
source: file.js

依赖模块：meta

mkdocs支持(实际是python markdown支持)的元信息内容，会被自动转为对应的html，目前支持以下4种meta


    title: Lorem ipsum dolor sit amet
    description: Nullam urna elit, malesuada eget finibus ut, ac tortor.
    path: path/to/file
    source: file.js

注意:

1. 要写在markdown文档的最上方

2. 写完后要加一个空行

会被自动翻译成

    <head>
    <meta name="description" content="Nullam urna elit, malesuada eget finibus ut, ac tortor.">
    <title>Lorem ipsum dolor sit amet</title>
    </head>
    以及<body>中某个位置会看到<a href="/path/to/file/file.js" title="file.js" class="md-source-file">

其中path和source的效果:
