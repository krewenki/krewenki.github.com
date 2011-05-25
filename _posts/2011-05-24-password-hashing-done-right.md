---
layout: post
title: "Password Hashing Done Right"
permalink: /2011/05/password-hashing-done-right
---

After reading a very informative piece by <a href="http://codahale.com/how-to-safely-store-a-password/">Coda Hale</a> about the proper way to store a password, I thought i'd provide a bit of information on the topic to help drive his point home.

There are at least two important things to think about when generating and storing passwords for any application. The first is general password strength, and the other is how you store that password.  

## Strong Passwords

The first thing to consider when dealing with passwords is enforcing some general measure of password strength.  All of the salting and encryption in the world won't do you any good if you have users with passwords that are too short, easily guessable, etc.  By enforcing a reasonable minimum length, and some rules for the composition of the password, you can introduce your first line of defence against brute force and/or dictionary attacks.

My personal stance is that the most important rule to enforce is password length. The simple act of adding a character to a password makes a brute force attack take an order of magnitude longer.  8 characters takes longer to brute force than 7, and 9 takes longer than 8, etc.  I often default to 8 characters, so that's the example I will use here.

For the purpose of this example, I'm going to build on the assumption that all of my passwords are made up of the following characters:

	abcdefghijklmnopqrstuvwxyz
	ABCDEFGHIJKLMNOPQRSTUVWXYZ
	0123456789
	!@#$%^&*()_+-=?

It is important to note that you may have more available characters, and you may have less.  I'm going to base my example around these 77 characters.  Again, the more characters available, the stronger a potential password is.

Your passwords are now made up of a minimum of 8 characters with 77 possible characters in each position.  To calculate the number of total possibilities for this schema, the math is simple. Number of characters raised to the power of length of password, or more simply:

77 * 77 * 77 * 77 * 77 * 77 * 77 * 77 = 1,235,736,291,547,681

In plain english, that's 1.2 quadrillion.  Seems like a lot, doesn't it?  If you were a human trying to guess the password through brute force *00000000, 00000001, 00000002 etc* and you could type passwords at a rate of 1 per second, it would take you just under 40,000,000 years (without stopping for coffee or bathroom breaks!)

If your computer, on the other hand, was trying to guess your password by means of brute force, it would take substantially less time.  My laptop, as of May 2011, can generate roughly 1,300,000 guesses per second, and that's based on unoptimized node.js code running on an already busy macbook pro.  As Coda [pointed out](http://www.golubev.com/hashgpu.htm) , there are methods available currently (for around $2,000 USD) that will allow for the generation of 5,600,000,000 hashes (not just attempts, full hashes) per second.  Your 40 million human years is now broken down to 3 days before your 8 character password can be guessed.

The problem with relying on strength alone is that these numbers are going to change quickly, and leaving the onus on your users to come up with more and more unique/strong passwords is a horrible solution.  Strong passwords increase the number of required guesses, but now you need to slow down the guessing.

## Generating and Storing Password Hashes

Modern computers can generate hashes incredibly fast.  While the example of 5,600,000,000 per second was a very specialized (albeit cheap) solution,  your own computer can generate a standard MD5 hash very quickly.  When you loaded this article, some simple code was run in the background, and determined that your web browser was able to generate <span id="perSecond">0</span> hashes per second.  You would have made <span id="calcCount">0</span> attempts at an md5 hashed password in the time you spent reading this.  As technology improves, so will the number of attempts per second. The same specialized application that attained 5,600,000,000 attempts per second a year ago may be able to do 10,000,000,000 next year.

### What about SHA1, SHA256, or even SHA512

While methods like SHA512 are slower than MD5, they are not the answer.  SHA512 will increase in speed on the same path that MD5 does, eventually allowing for billions or even trillions of attempts per second.

## Enter bcrypt

Coda explains why to use bcrypt best with the following:

>How? Basically, it's slow as hell. It uses a variant of the Blowfish encryption algorithm's keying schedule, and introduces a work factor, which allows you to determine how expensive the hash function will be. Because of this, bcrypt can keep up with Moore's law. As computers get faster you can increase the work factor and the hash will get slower.
>
>How much slower is bcrypt than, say, MD5? Depends on the work factor. Using a work factor of 12, bcrypt hashes the password yaaa in about 0.3 seconds on my laptop. MD5, on the other hand, takes less than a microsecond.
>
>So we're talking about 5 or so orders of magnitude. Instead of cracking a password every 40 seconds, I'd be cracking them every 12 years or so. Your passwords might not need that kind of security and you might need a faster comparison algorithm, but bcrypt

Basically, you can specify the amount of *work* bcrypt performs to generate your hash.  As technology improves, you can increase the amount of work you do to derive your hashes, keeping up with the modern technology curve.

Until there is a massively parallel version of bcrypt that can benefit from CUDA/OpenCL, this method moves brute forcing your hashes back into the thousands/millions of years range.  I'd say that is about as "difficult to brute force" as anyone can currently ask for.

## Closing Thoughts

While this focuses primarily on the generation and hashing of passwords, and is meant to deal with ensuring the security of your passwords should your password database be compromised, it is not all you need to focus on when securing your password data.  There are a host of other methods that can and should be used to ensure proper password handling such as lockouts, resets, etc.  Other security measures can be covered in a separate post. This one covers the important thing:

Use bcrypt


<script>
/*
 * A JavaScript implementation of the RSA Data Security, Inc. MD5 Message
 * Digest Algorithm, as defined in RFC 1321.
 * Version 2.2 Copyright (C) Paul Johnston 1999 - 2009
 * Other contributors: Greg Holt, Andrew Kepert, Ydnar, Lostinet
 * Distributed under the BSD License
 * See http://pajhome.org.uk/crypt/md5 for more info.
 */
var hexcase=0;function hex_md5(a){return rstr2hex(rstr_md5(str2rstr_utf8(a)))}function hex_hmac_md5(a,b){return rstr2hex(rstr_hmac_md5(str2rstr_utf8(a),str2rstr_utf8(b)))}function md5_vm_test(){return hex_md5("abc").toLowerCase()=="900150983cd24fb0d6963f7d28e17f72"}function rstr_md5(a){return binl2rstr(binl_md5(rstr2binl(a),a.length*8))}function rstr_hmac_md5(c,f){var e=rstr2binl(c);if(e.length>16){e=binl_md5(e,c.length*8)}var a=Array(16),d=Array(16);for(var b=0;b<16;b++){a[b]=e[b]^909522486;d[b]=e[b]^1549556828}var g=binl_md5(a.concat(rstr2binl(f)),512+f.length*8);return binl2rstr(binl_md5(d.concat(g),512+128))}function rstr2hex(c){try{hexcase}catch(g){hexcase=0}var f=hexcase?"0123456789ABCDEF":"0123456789abcdef";var b="";var a;for(var d=0;d<c.length;d++){a=c.charCodeAt(d);b+=f.charAt((a>>>4)&15)+f.charAt(a&15)}return b}function str2rstr_utf8(c){var b="";var d=-1;var a,e;while(++d<c.length){a=c.charCodeAt(d);e=d+1<c.length?c.charCodeAt(d+1):0;if(55296<=a&&a<=56319&&56320<=e&&e<=57343){a=65536+((a&1023)<<10)+(e&1023);d++}if(a<=127){b+=String.fromCharCode(a)}else{if(a<=2047){b+=String.fromCharCode(192|((a>>>6)&31),128|(a&63))}else{if(a<=65535){b+=String.fromCharCode(224|((a>>>12)&15),128|((a>>>6)&63),128|(a&63))}else{if(a<=2097151){b+=String.fromCharCode(240|((a>>>18)&7),128|((a>>>12)&63),128|((a>>>6)&63),128|(a&63))}}}}}return b}function rstr2binl(b){var a=Array(b.length>>2);for(var c=0;c<a.length;c++){a[c]=0}for(var c=0;c<b.length*8;c+=8){a[c>>5]|=(b.charCodeAt(c/8)&255)<<(c%32)}return a}function binl2rstr(b){var a="";for(var c=0;c<b.length*32;c+=8){a+=String.fromCharCode((b[c>>5]>>>(c%32))&255)}return a}function binl_md5(p,k){p[k>>5]|=128<<((k)%32);p[(((k+64)>>>9)<<4)+14]=k;var o=1732584193;var n=-271733879;var m=-1732584194;var l=271733878;for(var g=0;g<p.length;g+=16){var j=o;var h=n;var f=m;var e=l;o=md5_ff(o,n,m,l,p[g+0],7,-680876936);l=md5_ff(l,o,n,m,p[g+1],12,-389564586);m=md5_ff(m,l,o,n,p[g+2],17,606105819);n=md5_ff(n,m,l,o,p[g+3],22,-1044525330);o=md5_ff(o,n,m,l,p[g+4],7,-176418897);l=md5_ff(l,o,n,m,p[g+5],12,1200080426);m=md5_ff(m,l,o,n,p[g+6],17,-1473231341);n=md5_ff(n,m,l,o,p[g+7],22,-45705983);o=md5_ff(o,n,m,l,p[g+8],7,1770035416);l=md5_ff(l,o,n,m,p[g+9],12,-1958414417);m=md5_ff(m,l,o,n,p[g+10],17,-42063);n=md5_ff(n,m,l,o,p[g+11],22,-1990404162);o=md5_ff(o,n,m,l,p[g+12],7,1804603682);l=md5_ff(l,o,n,m,p[g+13],12,-40341101);m=md5_ff(m,l,o,n,p[g+14],17,-1502002290);n=md5_ff(n,m,l,o,p[g+15],22,1236535329);o=md5_gg(o,n,m,l,p[g+1],5,-165796510);l=md5_gg(l,o,n,m,p[g+6],9,-1069501632);m=md5_gg(m,l,o,n,p[g+11],14,643717713);n=md5_gg(n,m,l,o,p[g+0],20,-373897302);o=md5_gg(o,n,m,l,p[g+5],5,-701558691);l=md5_gg(l,o,n,m,p[g+10],9,38016083);m=md5_gg(m,l,o,n,p[g+15],14,-660478335);n=md5_gg(n,m,l,o,p[g+4],20,-405537848);o=md5_gg(o,n,m,l,p[g+9],5,568446438);l=md5_gg(l,o,n,m,p[g+14],9,-1019803690);m=md5_gg(m,l,o,n,p[g+3],14,-187363961);n=md5_gg(n,m,l,o,p[g+8],20,1163531501);o=md5_gg(o,n,m,l,p[g+13],5,-1444681467);l=md5_gg(l,o,n,m,p[g+2],9,-51403784);m=md5_gg(m,l,o,n,p[g+7],14,1735328473);n=md5_gg(n,m,l,o,p[g+12],20,-1926607734);o=md5_hh(o,n,m,l,p[g+5],4,-378558);l=md5_hh(l,o,n,m,p[g+8],11,-2022574463);m=md5_hh(m,l,o,n,p[g+11],16,1839030562);n=md5_hh(n,m,l,o,p[g+14],23,-35309556);o=md5_hh(o,n,m,l,p[g+1],4,-1530992060);l=md5_hh(l,o,n,m,p[g+4],11,1272893353);m=md5_hh(m,l,o,n,p[g+7],16,-155497632);n=md5_hh(n,m,l,o,p[g+10],23,-1094730640);o=md5_hh(o,n,m,l,p[g+13],4,681279174);l=md5_hh(l,o,n,m,p[g+0],11,-358537222);m=md5_hh(m,l,o,n,p[g+3],16,-722521979);n=md5_hh(n,m,l,o,p[g+6],23,76029189);o=md5_hh(o,n,m,l,p[g+9],4,-640364487);l=md5_hh(l,o,n,m,p[g+12],11,-421815835);m=md5_hh(m,l,o,n,p[g+15],16,530742520);n=md5_hh(n,m,l,o,p[g+2],23,-995338651);o=md5_ii(o,n,m,l,p[g+0],6,-198630844);l=md5_ii(l,o,n,m,p[g+7],10,1126891415);m=md5_ii(m,l,o,n,p[g+14],15,-1416354905);n=md5_ii(n,m,l,o,p[g+5],21,-57434055);o=md5_ii(o,n,m,l,p[g+12],6,1700485571);l=md5_ii(l,o,n,m,p[g+3],10,-1894986606);m=md5_ii(m,l,o,n,p[g+10],15,-1051523);n=md5_ii(n,m,l,o,p[g+1],21,-2054922799);o=md5_ii(o,n,m,l,p[g+8],6,1873313359);l=md5_ii(l,o,n,m,p[g+15],10,-30611744);m=md5_ii(m,l,o,n,p[g+6],15,-1560198380);n=md5_ii(n,m,l,o,p[g+13],21,1309151649);o=md5_ii(o,n,m,l,p[g+4],6,-145523070);l=md5_ii(l,o,n,m,p[g+11],10,-1120210379);m=md5_ii(m,l,o,n,p[g+2],15,718787259);n=md5_ii(n,m,l,o,p[g+9],21,-343485551);o=safe_add(o,j);n=safe_add(n,h);m=safe_add(m,f);l=safe_add(l,e)}return Array(o,n,m,l)}function md5_cmn(h,e,d,c,g,f){return safe_add(bit_rol(safe_add(safe_add(e,h),safe_add(c,f)),g),d)}function md5_ff(g,f,k,j,e,i,h){return md5_cmn((f&k)|((~f)&j),g,f,e,i,h)}function md5_gg(g,f,k,j,e,i,h){return md5_cmn((f&j)|(k&(~j)),g,f,e,i,h)}function md5_hh(g,f,k,j,e,i,h){return md5_cmn(f^k^j,g,f,e,i,h)}function md5_ii(g,f,k,j,e,i,h){return md5_cmn(k^(f|(~j)),g,f,e,i,h)}function safe_add(a,d){var c=(a&65535)+(d&65535);var b=(a>>16)+(d>>16)+(c>>16);return(b<<16)|(c&65535)}function bit_rol(a,b){return(a<<b)|(a>>>(32-b))};

var totalCalculated = 0;
var perSecond = 0;
var possibleCharacters = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z','0','1','2','3','4','5','6','7','8','9','!','@','$','%','^','&','*','(',')','-','=','_','+'];
var calculateBlock = function(){
	var start = new Date().getTime() / 1000;
	for(i=0;i<10000;i++){
		password = '';
		for(n=0;n<8;n++){
			password += possibleCharacters[Math.floor(Math.random()*possibleCharacters.length)]
		}
		hash = hex_md5(password)
		totalCalculated++;
	}
	var finish = new Date().getTime() / 1000;
	var totalTime = finish - start;
	perSecond = Math.floor(10000 * (1/totalTime));
	document.getElementById('perSecond').innerHTML = Math.floor(perSecond);
	setInterval(function(){ totalCalculated += perSecond; document.getElementById('calcCount').innerHTML = totalCalculated; },1000);
}

calculateBlock();


</script>