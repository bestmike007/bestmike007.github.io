---
title: Secure Data Transmission between Pure PHP and JavaScript using RSA
author: mike
layout: post
permalink: /2011/08/secure-data-transmission-between-pure-php-and-javascript-using-rsa/
comments: true
categories:
  - IT
tags:
  - JavaScript
  - PHP
  - RSA
---
During the last few days I was looking for an approach to secure data transmission between server written in PHP and browser that runs JavaScript. I failed to find a direct solution, however I made some adaption to some of the existing open source projects and built my own solution to this problem.

<!--more-->

## The Existing Projects

At the very beginning I found this: <http://www.ohdave.com/rsa/> It's quite good solution for JavaScript and I searched for the correlated PHP solution and I found this easily: <http://www.sematopia.com/2008/10/rsa-encrypting-in-javascript-and-decrypting-in-php/>. However, it uses the [Crypt_RSA package][1] with several dependencies and in some circumstances those packages would not be available. So I started to search for some pure PHP solution.

Then I found this one: <http://code.google.com/p/bi2php/>. I tried but failed to run this on my development computer with wamp running on Windows 2008r2. And this one: <http://stevish.com/rsa-encryption-in-pure-php>. I don't know because I hadn't tried to make it work because I failed to find a correlated JavaScript implementation. And I found this from stanford: <http://www-cs-students.stanford.edu/~tjw/jsbn/>. This is quite good except for the 'message too long' limitation. And <https://ziyan.info/exibits/javascript-rsa/login.html> is almost based on that. At last I noticed <http://phpseclib.sourceforge.net/>. I'm quite sure it's what I'm looking for and I tried to adapt it to the projects listed above but they all failed.

Then I tried to make some adaption for some of the JavaScript implementation and finally I built an adapted version of jsbn that works with phpseclib.

## The Solution

This solution makes phpseclib works with RSA and ECC in JavaScript from <http://www-cs-students.stanford.edu/~tjw/jsbn/> with some modification.

### 1. Edit the rsa.js, add a function after the RSAEncrypt function:

<pre><code class="language-javascript">function RSAEncryptLong(text) {
  var length = ((this.n.bitLength()+7)&gt;&gt;3) - 11;
  if (length &lt;= 0) return false;
  var ret = "";
  var i = 0;
  while(i + length &lt; text.length) {
    ret += this._short_encrypt(text.substring(i,i+length));
    i += length;
  }
  ret += this._short_encrypt(text.substring(i,text.length));
  return ret;
}</code></pre>

### 2. modify the last line from

<pre><code class="language-javascript">RSAKey.prototype.encrypt = RSAEncrypt;</code></pre>

to:

<pre><code class="language-javascript">RSAKey.prototype._short_encrypt = RSAEncrypt;
RSAKey.prototype.encrypt = RSAEncryptLong;</code></pre>

### 3. When using phpseclib to decrypt the message, set the encrypt mode to PKCS1:

<pre><code class="language-php">$rsa = new Crypt_RSA();
$rsa-&gt;setEncryptionMode(CRYPT_RSA_ENCRYPTION_PKCS1);</code></pre>

##  To Adopt the Solution

### 1. encrypt the message from browser:

<pre><code class="language-javascript">&lt;script type="text/javascript" src="jsbn.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="prng4.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="rng.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="rsa.js"&gt;&lt;/script&gt;
&lt;script&gt;
function encrypt(msg) {
	var rsa = new RSAKey();
	rsa.setPublic('8a5f4d4fa7dd78ca8539ba8b9581b30c9ce04e1998cd881d5279221984bc606e2c7d3368dc184b357507966a0f20930ba665cd9e914d6b0b67c8636ffe8cacfd', '10001');
	return rsa.encrypt(msg);
}
&lt;/script&gt;</code></pre>

### 2. decrypt the message from the server using PHP:

<pre><code class="language-php">require_once('Crypt/RSA.php');
define("KEY_PRIVATE", "-----BEGIN RSA PRIVATE KEY-----
MIIBOQIBAAJBAIpfTU+n3XjKhTm6i5WBswyc4E4ZmM2IHVJ5IhmEvGBuLH0zaNwYSzV1B5ZqDyCT
C6ZlzZ6RTWsLZ8hjb/6MrP0CAwEAAQJAAlK9TTln9No5nbwtvHHesWHaO5V0b6b5ubkXmHlrtuwR
nnNLGT9wqtIyP830/njo3qMFSIFKYGIErt+bSxEgBQIhAK5LTM2u2AudTUb6l1pi8qypXf7UHGUQ
bTxqPZaeh4gHAiEAyz0Wt0emBEieUDw7D4g3IXCb36cJcqDJ0OOz9rAwedsCIEV7QzzjrMDEjp/z
Gg8wTunCAvSpfkBT0hg5ih/XRtRVAiAQXVnf5iADBknhEgh7Zq9xvNyANLX5CeNWM4+BFIzCswIg
dGr1KW1fmIGJXoJ8qbFUbY7Bgk+cEc0kf2GvudfGQ5k=
-----END RSA PRIVATE KEY-----");

function decrypt(msg) {
	$rsa = new Crypt_RSA();
	$rsa-&gt;setEncryptionMode(CRYPT_RSA_ENCRYPTION_PKCS1);
	$rsa-&gt;loadKey(KEY_PRIVATE, CRYPT_RSA_PRIVATE_FORMAT_PKCS1);
	$s = new Math_BigInteger(msg, 16);
	retrun $rsa-&gt;decrypt($s-&gt;toBytes());
}</code></pre>

### 3. generate a new key pair:

<pre><code class="language-php">require_once('Crypt/RSA.php');
define('CRYPT_RSA_MODE', CRYPT_RSA_MODE_INTERNAL);
$rsa = new Crypt_RSA();
$rsa-&gt;setPublicKeyFormat(CRYPT_RSA_PUBLIC_FORMAT_RAW);
$key = $rsa-&gt;createKey(512);
echo $key['privatekey'];
echo "\n";
$e = new Math_BigInteger($key['publickey']['e'], 10);
$n = new Math_BigInteger($key['publickey']['n'], 10);
echo "Public Key:\n";
echo $e-&gt;toHex();
echo "\n";
echo $n-&gt;toHex();</code></pre>

At last is an example:

<pre><code class="language-php">&lt;?php
require_once('Crypt/RSA.php');
define("KEY_PRIVATE", "-----BEGIN RSA PRIVATE KEY-----
MIIBOQIBAAJBAIpfTU+n3XjKhTm6i5WBswyc4E4ZmM2IHVJ5IhmEvGBuLH0zaNwYSzV1B5ZqDyCT
C6ZlzZ6RTWsLZ8hjb/6MrP0CAwEAAQJAAlK9TTln9No5nbwtvHHesWHaO5V0b6b5ubkXmHlrtuwR
nnNLGT9wqtIyP830/njo3qMFSIFKYGIErt+bSxEgBQIhAK5LTM2u2AudTUb6l1pi8qypXf7UHGUQ
bTxqPZaeh4gHAiEAyz0Wt0emBEieUDw7D4g3IXCb36cJcqDJ0OOz9rAwedsCIEV7QzzjrMDEjp/z
Gg8wTunCAvSpfkBT0hg5ih/XRtRVAiAQXVnf5iADBknhEgh7Zq9xvNyANLX5CeNWM4+BFIzCswIg
dGr1KW1fmIGJXoJ8qbFUbY7Bgk+cEc0kf2GvudfGQ5k=
-----END RSA PRIVATE KEY-----");
function decrypt($msg) {
$rsa = new Crypt_RSA();
$rsa-&gt;setEncryptionMode(CRYPT_RSA_ENCRYPTION_PKCS1);
$rsa-&gt;loadKey(KEY_PRIVATE, CRYPT_RSA_PRIVATE_FORMAT_PKCS1);
$s = new Math_BigInteger($msg, 16);
return $rsa-&gt;decrypt($s-&gt;toBytes());
}
if(isset($_POST['submit'])) {
echo decrypt($_POST['enc_text']);
}
?&gt;
&lt;script type="text/javascript" src="jsbn.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="prng4.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="rng.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript" src="rsa.js"&gt;&lt;/script&gt;
&lt;script&gt;
function encrypt() {
var rsa = new RSAKey();
rsa.setPublic('8a5f4d4fa7dd78ca8539ba8b9581b30c9ce04e1998cd881d5279221984bc606e2c7d3368dc184b357507966a0f20930ba665cd9e914d6b0b67c8636ffe8cacfd', '10001');
document.getElementById('enc_text').value = rsa.encrypt(document.getElementById('plaintext').value);
}
&lt;/script&gt;
&lt;form action="" method="post"&gt;
Plain Text:&lt;br/&gt;
&lt;input id='plaintext' type="text" size="40" value="test"/&gt;&lt;br/&gt;
&lt;input type="button" onclick="encrypt()" value="Encrypt"/&gt;&lt;br/&gt;
Encrypted Text:&lt;br/&gt;
&lt;input id="enc_text" name='enc_text' type="text" size="40"/&gt;&lt;br/&gt;
&lt;input name="submit" type="submit" value="Submit" size="10"/&gt;
&lt;/form&gt;</code></pre>

## Finally

I hope this solution would help some of you guys. Feel free to ask me questions, and if there's something wrong with my article please help me to correct it.

 [1]: http://pear.php.net/package/Crypt_RSA/