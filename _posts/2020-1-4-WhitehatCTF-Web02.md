---
layout: post
title: WhitehatCTF - Web02
excerpt: "This was the second web challenge we solved in Whitehat I thought this problem was much more interesting than the first. "
categories: [Writeups, WhitehatCTF'19]
tags: [web]
---

This was the second web challenge we solved in Whitehat I thought this problem was much more interesting than the first. 

#### Web02
```
Challenge is available at http://52.78.36.66:81/
```

When we first visit the website, we see there is a form we can upload a picture which will get randomly hashed. It is restricted to being a PNG or JPG, we didn't manage to upload an SVG. There is also another section which allows you to make a post using a picture stored on the server.

The index page contained an HTML comment saying `<!-- flag is located at /flag.txt -->`. Now we knew what we wanted to find.

After looking at the requests we made when making a post, we noticed that we were making a post request to `http://52.78.36.66:81/post.php` 

If we visit this url in our browser with a get request, we are given the source of the page.

```php
<?php  
require_once('header.php');  
require_once('db.php');  
  
$post = [];  
  
function rotate_feature_img($post_id, $degrees) {  
	$post = get_post($post_id);  
	$src_file = $post['feature_img'];  
	$ext = strtolower(pathinfo($src_file, PATHINFO_EXTENSION));  
  
	$src_file = "uploads/" . $src_file;  
	if ( !file_exists( $src_file ) ) {  
		$base_url = "http://" . $_SERVER['HTTP_HOST'];  
		$src = $base_url. "/" . $src_file;  
	} else {  
		$src = $src_file;  
	}  
	try {  
		$img = new Imagick($src);  
		$img->rotateImage('white', $degrees);  
	} catch (Exception $e) {  
		die('fail to rotate image');  
	}  
	@mkdir(dirname($src_file));  
	@$img->writeImage($src_file);  
}  
  
function get_title() {  
	global $post;  
	echo $post['title'];  
}  
  
function get_body() {  
	global $post;  
	echo $post['body'];  
}  
  
function get_img_src() {  
	global $post;  
	$base_url = "http://" . $_SERVER['HTTP_HOST'];  
	echo $base_url."/uploads/".$post['feature_img'];  
}  
  
function check_theme($theme) {  
	if(!in_array($theme, scandir('./themes'))) {  
	die("Invalid theme!");  
	}  
}  
  
if (isset($_GET['post_id'])) {  
	require_once('post_header.php');  
	  
	$post_id = $_GET['post_id'];  
	global $post;  
	$post = get_post($post_id);  
	if (isset($_GET['theme'])) {  
	$theme = $_GET['theme'];  
	} else {  
	$theme = 'theme1.php';  
	}  
	check_theme($theme);  
	include('themes/'.$theme);  
	echo("<button class='btn btn-primary' id='edit-btn'>Edit image</button>  
	<form action='post.php' method='post' width='50%'>  
	<div class='form-group' id='edit-div'>  
	<label for='exampleInputEmail1'>Degree</label>  
	<input type='text' class='form-control' id='exampleInputEmail1' placeholder='90' name='degree'>  
	<input type='hidden' class='form-control' id='exampleInputEmail2' value='$post_id' name='post_id'>  
	<button type='submit' class='btn btn-primary'>Rotate</button>  
	</div>");  
	require_once('post_footer.php');  
}  
else if (isset($_POST['post_id']) && isset($_POST['degree'])) {  
	$post_id = $_POST['post_id'];  
	rotate_feature_img($post_id, (int) $_POST['degree']);  
	header("Location: /post.php?post_id=$post_id&theme=theme1.php");  
} else {  
	show_source('post.php');  
}  
  
require_once('footer.php');  
?>`
```

I also noticed that the name of the website was `Sordpress`, similar to wordpress. I looked for similar wordpress exploits. I found an [article](https://blog.ripstech.com/2019/wordpress-image-remote-code-execution/) which explained something similar. 

Looking back at the source,  it is obvious that `"http://" . $_SERVER['HTTP_HOST']` can be exploited. On a server we control, we uploaded a file that we wanted to be uploaded to the challenge server. The image we used was created using the following command:
`exiftool -Comment='<?php include "/flag.txt";?>' flag.png`
This would put the php code in plaintext into the flag.png file. Other payloads we tried did not work. 

The current process to upload our own files was the following.
1. Upload our exploited png to our own server
2. Make a new post using the website
	* `HTTP_HOST` was set to the ip of our server
	* Featured image was set to `flag.png`

Now we had our file uploaded to the server, what could we do with it? We determined there was not much. It seemed like all files ending with `.php` were forbidden. However, when looking at the wordpress article, it mentioned putting the file into the themes folder. An observation we made was that when looking at a post, there was a parameter `theme`. We first thought this could be an LFI but when looking at the source, it seemed like LFI was impossible as it verified the file was in the `/themes` directory.  But, what if we could upload our file and put it into the themes folder. Then we could include it without problems and get our flag. 

The final process to get the flag was the following.
1. Upload our exploited png to our own server and put it in `/themes/` folder on a server we controlled
2. Make a new post using the website
	* `HTTP_HOST` was set to the ip of our server
	* Featured image was set to `../themes/flag.png`
		* The `../` was used to get rid of the `/uploads` which was prepended to our URL
3. Rotate the image, allowing imagemagick to fetch the image from our server and write it in the `themes` folder.
4.  Finally, we could include our uploaded file using the themes argument by visiting `http://52.78.36.66:81/post.php?post_id=VBtLDmQP&theme=flag.png`
