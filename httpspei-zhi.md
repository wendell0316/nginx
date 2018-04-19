安装 rpm

rpm -ivh

user root;

worker\_processes    auto

location ^~ /api/spl {

```
rewrite '^/api/spl\(.\*\)' $1;

expires off;

proxy\_pass http://spl\_server;

proxy\_set\_header Host              $host;

proxy\_set\_header X-Real-IP         $remote\_addr;

proxy\_set\_header X-Forwarded-For   $proxy\_add\_x\_forwarded\_for;

proxy\_set\_header X-Forwarded-Host  $host:$server\_port;

proxy\_set\_header X-Forwarded-Proto $scheme;

break;
```

}

location ^~ /api/logtrace {

```
rewrite "^/api\(.\*\)" $1;

expires off;

proxy\_pass http://bigdata\_server;

proxy\_set\_header Host              $host;

proxy\_set\_header X-Real-IP         $remote\_addr;

proxy\_set\_header X-Forwarded-For   $proxy\_add\_x\_forwarded\_for;

proxy\_set\_header X-Forwarded-Host  $host:$server\_port;

proxy\_set\_header X-Forwarded-Proto $scheme;

break;
```

}

location ^~ /api/bigdata {

```
rewrite "^/api/bigdata\(.\*\)" $1;

expires off;

proxy\_pass http://bigdata\_server;

proxy\_set\_header Host              $host;

proxy\_set\_header X-Real-IP         $remote\_addr;

proxy\_set\_header X-Forwarded-For   $proxy\_add\_x\_forwarded\_for;

proxy\_set\_header X-Forwarded-Host  $host:$server\_port;

proxy\_set\_header X-Forwarded-Proto $scheme;

break;
```

}

location ^~ /api/topology {

```
rewrite "^/api\(.\*\)" $1;

expires off;

proxy\_pass http://topology\_server;

proxy\_set\_header Host              $host;

proxy\_set\_header X-Real-IP         $remote\_addr;

proxy\_set\_header X-Forwarded-For   $proxy\_add\_x\_forwarded\_for;

proxy\_set\_header X-Forwarded-Host  $host:$server\_port;

proxy\_set\_header X-Forwarded-Proto $scheme;

break;
```

}

  location ~\* \.\(otf\|eot\|woff\|ttf\|woff2\)$ {

    types     {font/opentype otf;}

    types     {application/vnd.ms-fontobject eot;}

    types     {font/truetype ttf;}

    types     {application/font-woff woff;}

    types     {font/woff2 woff2;}

  }



  location ~\* \.\(?:jpg\|jpeg\|gif\|png\|ico\|cur\|gz\|svg\|svgz\|mp4\|ogg\|ogv\|webm\|htc\|ttf\|woff\|woff2\|css\|js\|map\)$ {

    access\_log off;

    expires max;

    more\_set\_headers "Cache-Control: public, immutable";

    break;

  }

