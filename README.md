# Directory-Traversal-Cheat-Sheet

Directory traversal (also known as file path traversal) is a web security vulnerability that allows an attacker to read arbitrary files on the server that is running an application.

Consider the following URL: <strong><em>randomwebsite111.com/loadImage?filename=cutekitty18.png</em></strong>

The loadImage URL takes a filename parameter and returns the contents of the specified file. The image files themselves are stored on disk in the location <strong>/var/www/images/</strong>. To return an image, the application appends the requested filename to this base directory and uses a filesystem API to read the contents of the file. In the above case, the application reads from the following file path: <strong>/var/www/images/cutekitty18.png</strong>

The application implements no defenses against directory traversal attacks, so an attacker can request the following URL to retrieve an arbitrary file from the server's filesystem:

<strong><em>randomwebsite111.com/loadImage?filename=../../../etc/passwd</em></strong>

This causes the application to read from the following file path:

<strong>/var/www/images/../../../etc/passwd</strong>

On Windows, both ../ and ..\ are valid directory traversal sequences, and an equivalent attack to retrieve a standard operating system file would be:
<strong><em>randomwebsite111.com/loadImage?filename=..\..\..\windows\win.ini</em></strong>


<h1>Cheat Sheet:</h1>


You might be able to use an absolute path from the filesystem root, such as <strong>filename=/etc/passwd</strong>, to directly reference a file without using any traversal sequences.

You might be able to use nested traversal sequences, such as <strong>....// or ....\/</strong>

You might be able to use various non-standard <strong>encodings</strong>:

<ul>
  <li> . = %2e</li>
  <li> / = %2f</li>
  <li> \ = %5c</li>
</ul> 

<ul>
  16-bit:
  <li> . = %u002e</li>
  <li> / = %u2215</li>
  <li> \ = %u2216</li>
</ul>

<ul>
  Double URL:
  <li> . = %252e</li>
  <li> / = %252f</li>
  <li> \ = %255c</li>
</ul>

<ul>
  UTF-8:
  <li> . = %c0%2e, %e0%40%ae, %c0ae</li>
  <li> / = %c0%af, %e0%80%af, %c0%2f</li>
  <li> \ = %c0%5c, %c0%80%5c</li>
</ul>

example: <em>..%c0%af or ..%252f</em>

If an application requires that the user-supplied filename must end with an expected file extension, such as <strong>.png</strong>, then it might be possible to use a <strong>null byte</strong> to effectively terminate the file path before the required extension. 

For example:
<strong>filename=../../../etc/passwd%00.png</strong>

<h1>Prevention</h1>
<p>The most effective way to prevent file path traversal vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior in a safer way.

If it is considered unavoidable to pass user-supplied input to filesystem APIs, then two layers of defense should be used together to prevent attacks:</p>
<ul>
  <li>The application should validate the user input before processing it. Ideally, the validation should compare against a whitelist of permitted values. If that isn't possible for the required functionality, then the validation should verify that the input contains only permitted content, such as purely alphanumeric characters.</li>
  <li>After validating the supplied input, the application should append the input to the base directory and use a platform filesystem API to canonicalize the path. It should verify that the canonicalized path starts with the expected base directory.</li>
</ul>

<h5>Example of some simple Java code to validate the canonical path of a file based on user input:</h5>

    $File file = new File(BASE_DIRECTORY, userInput);
    if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
    // process file
    }
    
 
<h1>Credits</h1>
<ul>
  <li>https://portswigger.net/web-security/</li>
  <li>https://www.owasp.org/index.php/Path_Traversal</li>
  <li>https://github.com/kobs0N/</li>
</ul>
