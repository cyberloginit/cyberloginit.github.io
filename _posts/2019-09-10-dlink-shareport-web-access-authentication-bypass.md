---
layout: post
title: D-Link SharePort Web Access Authentication Bypass
---

# D-Link SharePort Web Access Authentication Bypass

## Vulnerability Description

SharePort Web Access is a feature available to most D-Link wireless home routers with a USB port.

It is enabled by default.

![default_config_file](/images/default_config_file.png)

Tutorial  from its [website](https://eu.dlink.com/uk/en/support/faq/routers/mydlink-routers/dir-880l/how-do-i-configure-shareport-web-access-on-my-router) shows that just insert your USB Flash Drive into your router, then you will be able to access it from the Internet by visit a URL like `http://[ip]:8181`.

Theoretically you have to be authorized by entering your router’s administrator username and password.

However, D-Link screws it up pretty badly.

**First**: If you have not changed them from the defaults, the username should be Admin, and the password field should be left **blank**.

**Second**: The authentication can be bypassed directly. Just visit `folder_view.php` or `category_view.php` or any valid page in this path and authentication is not required, including even `logininfo.xml`.

In some cases, visit `http://[ip]:[port]/webaccess/folder_view.php` instead `http://[ip]:[port]/folder_view.php`.

![web_access_directory](/images/web_access_directory.png)

## Vulnerability Scope

At least these wireless router models below are affected, each with its latest firmware installed.

|  Model   | Revision | Firmware |  Note  |
| :------: | :------: | :------: | :----: |
| DIR-868L |   REVB   |   2.03   | Latest |
| DIR-885L |   REVA   |   1.20   | Latest |
| DIR-895L |   REVA   |   1.21   | Latest |


## Vulnerability Analysis

So, how the authentication fails ？

Take `http://[ip]:[port]/folder_view.php` for example.

### Authentication Functions

`folder_view.php` is actually a HTML with JavaScript, nothing PHP here.

On line 719

```
<body onLoad="load_value();get_login_info()">
```

`get_login_info()` is supposed to do the authentication .

![call_flow](/images/call_flow.png)

`get_login_info()` 

```js
function get_login_info(){
    var xml_request = new XMLRequest(get_login_info_result);
    var para = "request=get_login_info";

    xml_request.exec_webfile_cgi(para);
}
```

`XMLRequest()`

```js
/**
 * XMLRequest( ) - Constructor for building XMLRequest Object 
 
 *	Parameter(s) :
 * 	onReqComp : the function that you want to perform after receive response from the web server
 *
 * Variable(s) :
 * 	http_req   :  user custom function for performing post response.
 * 	onReqComp  :  user custom function for performing post response.
 *
 **/
function XMLRequest(onReqComp){
	this.http_req = create_http_request();	
	this.onReqComp = onReqComp;	
}

/**
 * XMLRequest.prototype - Prototype of XMLRequest Object 
 *
 * Data members:
 * 	READY_STATE_***:  ready state constants
 *
 * Methods:
 * 	loading_xml()		:  get remote xml document
 * 	exec_cgi()			:  requesting server performing CGI command
 *  	get_login_level() :  get user's login level
 * 	onReadyState()		:  callback function while readystate changing
 **/
XMLRequest.prototype = {

READY_STATE_UNINITIALIZED 	: 0,
READY_STATE_LOADING 			: 1,
READY_STATE_LOADED 			: 2,
READY_STATE_INTERACTIVE 	: 3,
READY_STATE_COMPLETE 		: 4,


loading_xml: function(which_url){	
	//...
},

exec_cgi: function(para){
	//...			
},

exec_webfile_cgi: function(para){
	var req_url = "logininfo.xml?" + para;
	var obj = this;
		
	if (this.http_req){	
		this.http_req.onreadystatechange = function() {obj.onReadyState.call(obj)};				
		this.http_req.open('GET', req_url, true);
		this.http_req.setRequestHeader('Content-length', para.length);	
		this.http_req.setRequestHeader("Connection", "close");
		this.http_req.send(null);		
		
		return true;
	}			
	
	return false;			
},

exec_auth_cgi: function(para){
	//...	
},

json_cgi: function(para){
	//...		
},

get_login_level: function(){	
	//...
},

onReadyState: function(){
	//...
}
}
```

`get_login_info_result()`

```js
function get_login_info_result(http_req){
	//...
	var my_xml = http_req.responseXML;

	if (check_user_info(my_xml.getElementsByTagName("redirect_page")[0])){

		storage_user.put("id", get_node_value(my_xml, "user_name"));
		storage_user.put("tok", get_node_value(my_xml, "user_pwd"));

		load_webfile_settings();
	}
}
```

`check_user_info()`

```js
/**
 *	check_user_info() : to check if the user has login yet
 *
 *	Parameter(s) :
 *		redirect_page : a XML's element object which contains a redirect page
 *
 * Return :	True or False
 * 	
 **/
function check_user_info(redirect_page){
	var which_page;
	
	if (redirect_page != null){	
		which_page = redirect_page.firstChild.nodeValue;
		location.href = html_obj.get_value(which_page);				
		return 0;
	}
	
	return 1;
}
```
### How It Failed

When `get_login_info()` `GET`

```
http://[ip]:[port]/logininfo.xml?request=get_login_info
```

We get

```xml

<?xml version="1.0"?>
<root>
	<user_name>admin</user_name>
	<user_pwd>t</user_pwd>
	<volid>1</volid>
</root>
```

In `get_login_info_result()`

```
var my_xml = http_req.responseXML;
```

`my_xml` is actually the XML content above.

`my_xml.getElementsByTagName("redirect_page")` should return

```
HTMLCollection []
length: 0
__proto__: HTMLCollection
```

And if we should try to take its first element, `undefined` is returned.

In `get_login_info_result()`

```js
if (check_user_info(my_xml.getElementsByTagName("redirect_page")[0])){

	storage_user.put("id", get_node_value(my_xml, "user_name"));
	storage_user.put("tok", get_node_value(my_xml, "user_pwd"));

	load_webfile_settings();
}
```

If `check_user_info(my_xml.getElementsByTagName("redirect_page")[0])` is `True`, `load_webfile_settings()` will be executed, which means user is authorized to visit this page `folder_view.php`.

However, in `check_user_info()`

```
function check_user_info(redirect_page){
	var which_page;
	
	if (redirect_page != null){	
		which_page = redirect_page.firstChild.nodeValue;
		location.href = html_obj.get_value(which_page);				
		return 0;
	}
	
	return 1;
}
```

If element `redirect_page = my_xml.getElementsByTagName("redirect_page")[0]`  does not exist, **1** is returned, and user is authorized to proceed !

## Summary

The authentication procedure for D-Link SharePort Web Access basically fails at every aspects.
1. Frontend authentication is not very reliable;
2. Name your variable wisely, or you will most likely pay for it.

## Reference
* [https://eu.dlink.com/uk/en/support/faq/routers/mydlink-routers/dir-880l/how-do-i-configure-shareport-web-access-on-my-router](https://eu.dlink.com/uk/en/support/faq/routers/mydlink-routers/dir-880l/how-do-i-configure-shareport-web-access-on-my-router)
