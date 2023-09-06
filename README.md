# hugoBasicExample

This repository offers an example site for [Hugo](https://gohugo.io/) and also it provides the default content for demos hosted on the [Hugo Themes Showcase](https://themes.gohugo.io/).

# Using

1. [Install Hugo](https://gohugo.io/overview/installing/)
2. Clone this repository
```bash
git clone https://github.com/gohugoio/hugoBasicExample.git
cd hugoBasicExample
```
3. Clone the repository you want to test. If you want to test all Hugo Themes then follow the instructions provided [here](https://github.com/gohugoio/hugoThemes#installing-all-themes)
4. Run Hugo and select the theme of your choosing
```bash
hugo server -t YOURTHEME
```
5. Under `/content/` this repository contains the following:
- A section called `/post/` with sample markdown content
- A headless bundle called `homepage` that you may want to use for single page applications. You can find instructions about headless bundles over [here](https://gohugo.io/content-management/page-bundles/#headless-bundle)
- An `about.md` that is intended to provide the `/about/` page for a theme demo
6. If you intend to build a theme that does not fit in the content structure provided in this repository, then you are still more than welcome to submit it for review at the [Hugo Themes](https://github.com/gohugoio/hugoThemes/issues) respository

拉取的时候:git clone https://github.com/jayzhouxl/online_web.git --recurse-submodules(拉去子模块)
在gitlod上执行的命令:hugo server -D -F --baseURL $(gp url 1313) --liveReloadPort=443 --appendPort=false --bind=0.0.0.0 --themesDir ../..
不然会显示异常,这里主要是绑定了url,然后里面的一些标签的url命令会指定到这边

网站的浏览器小图标如何修改:
1.修改位置: hugo-clarity/layouts/partials/favicon.html(  3 {{- $favicon := absURL (printf "%s%s" $iconsDir "z.png" ) }})
2.将所需的图片移到对应的目录中:themes/hugo-clarity/static/icons
网站的标题修改:
./config/_default/languages.toml
修改网站头的颜色和夜晚的背景色:
vim ./themes/hugo-clarity/assets/sass/_variables.sass 这上面两个bg所指的地方
3.免责声明的修改:目前免责声明是放在侧边栏的，目前是关闭状态，如果打开的话，可以在这个(vim themes/hugo-clarity/layouts/partials/sidebar.html)修改免责声明的位置
