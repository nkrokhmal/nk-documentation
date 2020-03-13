Nginx
=====

LoadBalancing
-------------


Use Nginx's HTTP module to load balance over HTTP servers using `upstream` block. Example

.. code:: console

        upstream backend {
                server 10.10.10.10:80 weight=1;
                server example.com    weight=2;
        }
        server {
                location / {
                        proxy_pass http://backend;
                }
        }

The weight param instruct Nginx to pass twice as many connections to the second server, defaults `weight=1`.

TCP LoadBalancing
-----------------

If you need to distribute load between two or more TCP servers using `upsrteam` block

.. code:: console

        upstream {
                upstream mysql_read {
                        server read1.example.com:3306 wright=5;
                        server read2.example.com:3306;
                        server 10.10.12.34:3306    backup;
                }

                server {
                        listen 3306;
                        proxy_pass mysql_read;
                }
        }

