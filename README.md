# Using a proxy for web development

If you are working with a web site that exists on a production server and on a development server and possibly also a staging server, then you need a solution to switch between the different servers.

One solution is to use access the web site through different hosts name depending on which machine you want to work. For example, you can have www.example.org for the production site and dev.example.org for the development server. This, however, requires that your web site can manage different domain names. When developing a WordPress site that means you have to search the database for the production site domain name and replace it with the development site domain name every time you copy the production site to the development site.

Another approach is to add the web site domain to [the `hosts` file](https://en.wikipedia.org/wiki/Hosts_(file)#Location_in_the_file_system) of your local computer and change its IP number between the production server and development server depending on which one you want to work with.

But there is a third way which makes life much easier for you. Use a proxy. It requires a little bit of work to set it up, but then you almost can forget about it. Let me explain how.

## Create a PAC-file

First of all, you need to create a [*proxy auto-config* (*PAC*) file](https://developer.mozilla.org/en-US/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_(PAC)_file). It is a file with the JavaScript function `FindProxyForURL(url, host)` that returns a string with one or more access method specifications.

Following example returns the string `SOCKS5 127.0.0.1:8888; DIRECT` for any subdomain to `example.org` and `DIRECT` for every other domain.

```javascript
function FindProxyForURL(url, host) {

    if (dnsDomainIs(host, ".example.org"))
        return "SOCKS5 127.0.0.1:8888; DIRECT";

    return "DIRECT";

}
```

Create a file named `.pac` in your home directory and copy and paste the above code to it. Change `.example.org` to the domain of your production site.

Try to open it in your browser. Its URL is `file://<HOME>/.pac` where `<HOME>` is the absolute path to your home directory. It should look something similar to `file:///home/thomas` on Linux and `file:///User/thomas` on Mac OS X and `file:///c:/Users/thomas` on Windows.

## Firefox settings

Next step is to configure your web browser to use the PAC file from above. This is how that is done in Firefox:

1.  Open [Firefox preferences page](about:preferences).
1.  On the [*General*-tab](about:preferences#general), scroll down to the end where you find *Network Settings*. Click on the *Settings…* button. A dialogue box named *Connection Settings* opens.
1.  Select *Automatic proxy configuration URL* and enter the PAC file's URL from above in the field.
1.  Check the option *Proxy DNS when using SOCKS v5*.
1.  Click the *OK* button and close the preference page.

Notice that you still can browse the web as usual, including your production site.

## Configure the development server

On the development server, you need to add the domain of the web site to its hosts file and point it to localhost. If your server runs a Unix-like operating system (e.g. Ubuntu, FreeBSD and similar), you open the file `/etc/hosts` and make sure there is a line looking like this

    127.0.0.1 www.example.org

where `www.example.org` is replaced with the domain name of your production site. A space or a tab can separate several domain names.

## Start the SOCK5 proxy

Now, to access the development server instead of the production server, you only have to start a SOCK5 proxy. On computers running a Unix-like operative system (including Mac OS X) it's straightforward; start a terminal and enter:

```bash
ssh -nNTD 0.0.0.0:8888 <dev-server>
```

where `<dev-server>` is replaced with either the domain name or the IP-number of the server with the development site. On Windows, you can install [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) and visit the [proxy panel](https://the.earth.li/~sgtatham/putty/0.73/htmldoc/Chapter4.html#config-proxy) to accomplish the same thing.

You can now visit the web site as usual. Notice that you are visiting it on the development server and not on the production server. If not, you might restart the web browser.

On Unix-like operative system, you just hit `CTRL` and `C` in the terminal window, to terminate the proxy. On Windows, you quit PuTTY.

As soon as you stop the proxy, you will be back on the production site when visiting the site.

## Script

Users of Linux and Mac OS X can use the script below to easily start and stop proxies.

### Install the script

1. Save the script below as a file named `proxy`.
2. Review `DEFAULT_SERVER` and the other global variables of the script.
3. Give yourself right to execute the file: `chmod +x proxy`.
4. Move the script to a directory on your path: `mv proxy /usr/local/bin`.
5. Let it create the PAC file: `proxy reset`.

You are now ready to use the script.
