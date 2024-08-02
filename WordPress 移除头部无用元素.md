# WordPress 移除头部无用元素

文档编写的Wordpress版本为 `6.6.1`。

`wp_head()` 是 WordPress 核心提供的一个非常重要的函数，它在页面 `<head>` 部分执行，用于插入各种元数据、脚本、样式表链接以及其他必要的 HTML 标签。以下是 `wp_head()` 函数可能输出的内容：

**1. 元数据：**

- RSS feed 链接，允许用户订阅站点更新。
- 网站图标（favicon）。
- 站点生成器信息（默认情况下显示 WordPress 版本，但可以禁用）。

**2. 链接标签：**

- 引用外部 CSS 文件，包括主题和插件的样式表。
- 引用字体文件，如 Google Fonts 或其他自定义字体。
- 
**3. 脚本标签：**

- 引入 JavaScript 文件，包括主题和插件的脚本。
- 输出脚本执行前的必要代码，如条件语句或自定义脚本。

**4. SEO 相关信息：**

- Open Graph 标签，用于 Facebook 和其他社交网络分享优化。
- Twitter 卡片元数据，改善 Twitter 分享预览。
  
**5. 其他：**

- XML-RPC 元数据，用于支持远程发布和编辑。
- Webmention 支持，用于跨站评论。
- 自定义元数据，由主题或插件添加。
  
```php
/**
 * 源码位置: wp-includes/general-template.php:3052
 * 
 * 启动wp_head动作。
 *
 * See {@see 'wp_head'}.
 *
 * @since 1.2.0
 */
function wp_head() {...}
```

## 1. 移除 WordPress 中版本信息

```php
// 源码位置: wp-includes/general-template.php:5023
$gen = '<meta name="generator" content="WordPress ' . esc_attr( get_bloginfo( 'version' ) ) . '">';
```
WordPress自动添加版本号信息，在`<head>`区域，可以看到：

```html
<meta name="generator" content="WordPress 6.6.1" />
```
移除WordPress版本信息对于增强网站的安全性和减少潜在的攻击风险是有益的。攻击者可能会利用`特定版本`的WordPress中存在的已知漏洞来进行攻击，因此隐藏版本号可以降低这种风险。有些自动化工具会根据版本号来筛选目标。

移除代码如下：

```php
add_filter('the_generator', '__return_false');
```

如果想更彻底地移除所有版本号相关的信息，可以使用以下代码：

```php
// 去除wp中所有版本号
function remove_wp_version() {
    global $wp;
    if (isset($wp->query_vars['feed']) && !empty($wp->query_vars['feed'])) {
        return;
    }
    remove_action('wp_head', 'wp_generator');
}
add_action('init', 'remove_wp_version');
// 去除样式与脚本版本号
function remove_script_style_ver($src) {
    if (strpos($src, 'ver=') !== false)
        $src = remove_query_arg('ver', $src);
    return $src;
}
add_filter('script_loader_src', 'remove_script_style_ver', 15, 1);
add_filter('style_loader_src', 'remove_script_style_ver', 15, 1);
```

## 2. 去除 wp-embed 功能

虽然 `wp-embed` 提高了内容的丰富性和用户体验，但它也可能对页面性能产生负面影响，特别是当页面包含大量嵌入式媒体时。这是因为每个嵌入式内容都可能需要额外的 HTTP 请求和资源加载，这会增加页面的加载时间和服务器负载。

如果网站不需要使用嵌入式内容，或者为了提高页面加载速度，可以考虑禁用 `wp-embed`。这可以通过添加以下代码来实现：


```php
function disable_wp_embed() {
    // 移除 wp-embed.min.js 文件的加载
    wp_deregister_script('wp-embed');
}
add_action('wp_enqueue_scripts', 'disable_wp_embed', 100);

// 移除嵌入式发现链接
remove_action('wp_head', 'wp_oembed_add_discovery_links');
remove_action('wp_head', 'wp_oembed_add_host_js');
```

## 3. 移除 shortlink 标签

移除`shortlink`可以带来一定的安全性和性能优势，同时也可能简化 URL 结构和提升 SEO 效果。

在`<head>`中效果如下:
```html
<link rel='shortlink' href='http://jellydai.com/?p=701' />
```

```php
/**
 * 源码位置: wp-includes/link-template.php:4180
 * 
 * 如果为当前页面定义了短链接，则将rel=shortlink注入头部。
 *
 * Attached to the {@see 'wp_head'} action.
 *
 * @since 3.0.0
 */
function wp_shortlink_wp_head() {...}
/**
 * 源码位置: wp-includes/link-template.php:4197
 * 
 * 如果为当前页面定义了短链接，则发送链接：rel=shortlink标头。
 *
 * Attached to the {@see 'wp'} action.
 *
 * @since 3.0.0
 */
function wp_shortlink_header() {...}
```
移除代码如下：
```php
// 本质就是取消这两个函数执行
remove_action( 'wp_head', 'wp_shortlink_wp_head', 10, 0 );
remove_action( 'template_redirect','wp_shortlink_header',11,0);
```

## 4. 移除 RSD 与 Feed

**RSD (Really Simple Discovery)**

RSD 是一种 `XML` 格式的元数据文件，用于描述一个网站的 `XML-RPC` 接口，使得其他应用可以轻松地发现和使用这些接口。RSD 文件通常包含一个网站的 `XML-RPC` 地址，以及一些关于网站的信息，如名称、主页 URL 和博客 URL。RSD 的主要用途是方便博客编辑器和其他应用程序与博客进行交互，比如发布新文章、编辑现有文章等。

**Feed**

`Feed`（`RSS` 或 `Atom`）是一种让网站能够发布更新内容的格式，订阅者可以通过`RSS 阅读器`或类似工具接收这些更新。Feed 提供了一种标准化的方式来跟踪网站的新内容，而无需频繁访问网站本身。

** 移除 RSD 和 Feed 的考虑**

- **安全性**：移除 RSD 可以减少潜在的安全风险，因为 RSD 文件暴露了网站的 XML-RPC 接口，这可能成为攻击者的目标。同样，移除 Feed 可以防止攻击者通过 Feed URL 获取敏感信息或进行 DoS 攻击。
- **隐私**：移除 RSD 和 Feed 可以减少网站信息的公开程度，保护网站的隐私。
- **性能**：移除 RSD 和 Feed 可以减少 HTTP 请求，从而可能提高页面加载速度。

### 移除 RSD 的代码如下：

```php
/**
 * 源码位置：wp-includes/general-template.php:3378
 * 
 * 显示指向RSD服务终结点的链接。
 *
 * @link http://archipelago.phrasewise.com/rsd
 * @since 2.0.0
 */
function rsd_link() {...}
```

移除代码如下：

```php
remove_action('wp_head', 'rsd_link');
```

### 移除 Feed 的代码如下：

```php
/**
 * 源码位置：wp-includes/general-template.php:3100
 * 
 * 显示指向feed的链接.
 *
 * @since 2.8.0
 *
 * @param array $args Optional arguments.
 */
function feed_links( $args = array() ) {...}

/**
 * 源码位置：wp-includes/general-template.php:3157
 * 
 * 显示指向额外提要（如类别提要）的链接。
 *
 * @since 2.8.0
 *
 * @param array $args Optional arguments.
 */
function feed_links_extra( $args = array() ) {...}
```

移除代码如下：

```php
remove_action('wp_head', 'feed_links', 2);
remove_action('wp_head', 'feed_links_extra', 3);
```

## 5. 移除 wp-json 链接

`wp-json` 是 WordPress REST API 的入口点，它允许客户端通过 HTTP 请求与 WordPress 互动。通过移除 `wp-json` 链接，可以减少页面加载时间并提高网站性能。

```php
/**
 * 源码位置：wp-includes/rest-api.php:998
 * 
 * 将REST API链接标记输出到页头中。
 *
 * @since 4.4.0
 *
 * @see get_rest_url()
 */
function rest_output_link_wp_head() {...}
```

移除代码如下：

```php
remove_action( 'wp_head', 'rest_output_link_wp_head', 10, 0 );
```

## 6. 移除块编辑器样式、emoji样式和脚本

WordPress 默认情况下会加载一些 CSS 和 JavaScript 文件，这些文件主要用于增强用户体验，例如在编辑器中显示表情符号、添加块编辑器样式等。这些文件可以通过移除或禁用来减少页面加载时间。

> 移除全局样式或块编辑器样式，会影响`古腾堡`的样式, 但段落、标题等基础的块不受影响。

```php
// 移除全局样式
add_action( 'wp_enqueue_scripts', 'remove_global_styles' );
function remove_global_styles(){
	wp_dequeue_style( 'global-styles' );
}
// 移除块编辑器样式
function remove_wp_block_library_css(){
    wp_dequeue_style( 'wp-block-library' );
    wp_dequeue_style( 'wp-block-library-theme' );
}
add_action( 'wp_enqueue_scripts', 'remove_wp_block_library_css', 100 );
```

```php
// 移除emoji样式与脚本
remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
remove_action( 'wp_print_styles', 'print_emoji_styles' );
```

```php
// 移除经典编辑器样式
wp_deregister_style('classic-theme-styles');	
wp_dequeue_style('classic-theme-styles');
```

### 7. 效果展示

主题的index.php文件。

```php
<?php get_header()?> 
<main>
<?php the_content()?>
</main>
<?php get_footer()?> 
```

在不安装插件的情况，访客模式下访问，非常的干净整洁。

```html
<!doctype html>
<html lang="en-US">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="profile" href="https://gmpg.org/xfn/11">
<meta name='robots' content='noindex, nofollow' />
</head>
<body class="home blog wp-custom-logo"></body>
</html>
```

## 8. 代码合集

```php
/**
 * 移除 WordPress 默认块库 CSS
 */
function remove_wp_block_library_css(){
	// 移除 WordPress 块库 CSS
    wp_dequeue_style( 'wp-block-library' );
	// 移除 WordPress 块库主题 CSS
	wp_dequeue_style( 'wp-block-library-theme' );
    wp_dequeue_style( 'wp-block-library-theme' );
}
/**
 * 移除全局样式
 */
function remove_global_styles(){
	// 移除名为'global-styles'的样式队列
	wp_dequeue_style( 'global-styles' );
}
/**
 * 禁用 WordPress 内嵌脚本
 */
function disable_wp_embed() {
	// 取消注册 'wp-embed' 脚本
    wp_deregister_script('wp-embed');
}

/**
 * 移除经典主题样式
 */
function remove_classic_theme_styles(){
	// 取消注册经典主题样式，防止其被自动加载
	wp_deregister_style('classic-theme-styles');
	// 确保经典主题样式不会被加载到前端，即使之前已被注册
	wp_dequeue_style('classic-theme-styles');
}

/**
 * 移除脚本和样式中的版本号参数
 */
function remove_script_style_ver($src) {
    if (strpos($src, 'ver=') !== false)
        $src = remove_query_arg('ver', $src);
    return $src;
}
/**
 * 移除WordPress版本信息
 */
function remove_wp_version() {
    global $wp;
    if (isset($wp->query_vars['feed']) && !empty($wp->query_vars['feed'])) {
        return;
    }
    remove_action('wp_head', 'wp_generator');
}
/**
 * 清理WordPress头部
 * 只是去除头部的内容，例如api、feed、等功能还是存在的。
 */
function clean_head(){
    // 以下为添加动作或过滤器：需要编写对应的函数。
	// 去除wp中所有版本号
	add_action('init', 'remove_wp_version');
	// 去除样式与脚本版本号
	add_filter('script_loader_src', 'remove_script_style_ver', 15, 1);
	add_filter('style_loader_src', 'remove_script_style_ver', 15, 1);
	// 移除 wp-embed.min.js 文件的加载
	add_action('wp_enqueue_scripts', 'disable_wp_embed', 100);
    // 移除全局样式
	add_action( 'wp_enqueue_scripts', 'remove_global_styles' );
	// 移除区块样式
	add_action( 'wp_enqueue_scripts', 'remove_wp_block_library_css', 100 );
    // 移除经典编辑器
	add_action( 'wp_enqueue_scripts', 'remove_classic_theme_styles' );
    // 以下为移除动作：不需要编写函数，直接可以调用。
	// 移除嵌入式发现链接
	remove_action('wp_head', 'wp_oembed_add_discovery_links');
	remove_action('wp_head', 'wp_oembed_add_host_js');
	// 移除短链接	
	remove_action( 'wp_head', 'wp_shortlink_wp_head', 10, 0 );
	remove_action( 'template_redirect','wp_shortlink_header',11,0);
	// 移除RSD链接
	remove_action('wp_head', 'rsd_link');
	// 移除feed
	remove_action('wp_head', 'feed_links', 2);
	remove_action('wp_head', 'feed_links_extra', 3);
	// 移除REST API
	remove_action( 'wp_head', 'rest_output_link_wp_head', 10, 0 );
	// 移除emoji
	remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
	remove_action( 'wp_print_styles', 'print_emoji_styles' );

}
clean_head();
```