What is  a Cache?

A cache is a temporary storage area  where data can be stored for rapid access.

Why Cache PHP?

Compiling PHP every time a request comes in is a very resource intensive process and it is better to cache code rather than compile it every time. This not only reduces your  ram memory usage but also increases the speed of loading your php pages.

What are the Types of Cache available for PHP?

There are many but some of the prominent one's are Memcache, APC ( alternative PHP cache) , Zend cache.

Why APC?

Well cause it's an opcode cache that means it caches the compiled code, It is going to be included in PHP 6 and it is easy to configure.

How can I install?

Before you do add the below code to your php script to measure page load time

<?php echo get_num_queries(); ?> queries in <?php timer_stop(1); ?> seconds

You can add it to footer.php if you are using wordpress

You have to have PHP up and running if not then refer to my tutorial here and then install via yum package manager

$ yum install php-apc

How to configure?

There are two primary decisions to be made configuring APC. First, how much memory is going to be allocated to APC; and second, whether APC will check if a file has been modified on every request. The two ini directives that control these settings are apc.shm_size and apc.stat. You should also set apc.ttl low as that is time to live for the cache entries. The lower the better as when you cache is full the older entries will be flushed to make space for new, i f you have a higher ttl or a 0 ttl and you cache is full then the entire cache is flushed.

Open up the apc.ini config file and make the following changes

vim /etc/php/apc.ini
apc.ttl=3600
apc.stat=0
apc.shm_size=64M
cp /usr/share/doc/php-pecl-apc-3.1.15/apc.php  /var/www/html/

How can I clear the cache?

Add a file called apc_clear.php in your web root directory with the below code

               
 if (function_exists('apc_clear_cache') && $_GET['pass'] == 'secret') {
        if (apc_clear_cache() && apc_clear_cache('user'))
                print 'All Clear!';
        else
                print 'Clearing Failed!';
print_r(apc_cache_info());
} else {
 print 'Authenticate, please!';
}

What is the meaning of hits and misses?
PHP cache APC
Hits means the request was served from the cache and misses means it was not. You need to increase your cache memory incase if it get's  full often.

 

Source

http://css.dzone.com/articles/using-apc-correctly

http://ckon.wordpress.com/2012/01/02/speedup-php-opcode-cache-apc-xcache-eaccelerator/

http://linuxaria.com/howto/everything-you-need-to-know-about-apc-alternate-php-cache?lang=en

http://rtcamp.com/wordpress-nginx/tutorials/php/apc-cache-with-web-interface/

http://www.slideshare.net/benramsey/caching-with-memcached-and-apc
