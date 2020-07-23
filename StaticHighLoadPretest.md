# Выбор движка для async API python
MacBookPro 13 (2020) OsX Catalinia
virtualbox 1 CPU 1Gb: Ubuntu 20.04 Server 
python3.8.2 quart asyncio aiohttp uvloop

## 1. Quart статика
```
python3 01quart_static.py
```
```python
from quart import Quart
app = Quart(__name__)

@app.route('/')
async def hello():
    return 'Hello'
    
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5001)
```
##### Производительность 870 RPS 
```
wrk -t30 -c300 -d5s http://rec.local:5001/
  4429 requests in 5.10s, 610.52KB read
Requests/sec:    868.51
Transfer/sec:    119.72KB
```

## 2. Quart статика без логов
```
python3 01quart_nologging_static.py
```
```python
from quart import Quart
import logging

app = Quart(__name__)
@app.route('/')
async def hello():
    return 'Hello'
    
if __name__ == '__main__':
    logging.getLogger('quart.serving').setLevel(logging.ERROR)
    app.run(host="0.0.0.0", port=5002)
```

##### Производительность 1270 RPS 
```
wrk -t30 -c300 -d5s http://rec.local:5002/
  6478 requests in 5.10s, 0.87MB read
Requests/sec:   1270.17
Transfer/sec:    174.90KB
```

## 3. aiohttp web
```
python3 01aio_static.py
```
```python
from aiohttp import web

async def hello(request):
    return web.Response(text="Hello")    

app = web.Application()
app.add_routes([
    web.get('/', hello, name='default'),
])

if __name__ == '__main__':
    web.run_app(app, host='0.0.0.0', port=5003)
```

##### Производительность 3513 RPS 
```
wrk -t30 -c300 -d5s http://rec.local:5003/
  17917 requests in 5.10s, 2.65MB read
Requests/sec:   3513.06
Transfer/sec:    531.79KB
```
## 4. aiohttp web + uvloop
```
python3 01aio_uvloop_static.py
```
```python
from aiohttp import web
import asyncio
import uvloop
   
async def hello(request):
    return web.Response(text="Hello")    

asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
app = web.Application()
app.add_routes([
    web.get('/', hello, name='default'),
])

if __name__ == '__main__':
    web.run_app(app, host='0.0.0.0', port=5004)
```

##### Производительность 6178 RPS 
```
wrk -t30 -c300 -d5s http://rec.local:5004/
  31488 requests in 5.10s, 4.65MB read
Requests/sec:   6177.62
Transfer/sec:      0.91MB
```
## 5. NGINX static (for benchmark)
```
server {
    server_name rec.local;
    listen 80;
    root /var/www/rec.local;
    index index.html;                        
    location / {
        try_files $uri $uri/ =404;
    }
}
```

##### Производительность 11131 RPS 
```
wrk -t30 -c300 -d5s http://rec.local/
  56765 requests in 5.10s, 25.98MB read
  Socket errors: connect 0, read 0, write 0, timeout 22
Requests/sec:  11131.57
Transfer/sec:      5.10MB
```

## 1. Quart без логов, без reload и с uvloop
```
python3 01quart_maxtune_static.py
```
```python
from quart import Quart
import logging
import asyncio
import uvloop

asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
app = Quart(__name__)

@app.route('/')
async def hello():
    return 'Hello'

if __name__ == '__main__':
    logging.getLogger('quart.serving').setLevel(logging.ERROR)
    app.run(host="0.0.0.0", port=5010,debug=False,use_reloader=False)
```
##### Производительность 1724 RPS 
```
wrk -t30 -c300 -d5s http://rec.local:5010/
  8791 requests in 5.10s, 1.18MB read
Requests/sec:   1723.64
Transfer/sec:    237.34KB
```

# Вывод
Quart+uvloop 1724 RPS
aiohttp web + uvloop 6178 RPS
nginx static 11131 RPS
# Ограничения сравнения
- без https
- без pgsql
- без auth
- без полезной нагрузки
- локально, а не на VPS на сервере


