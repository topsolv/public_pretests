# Выбор движка для async API python
MacBookPro 13 (2020) OsX Catalinia
Virtualbox Ubuntu 20.04 server CPU:1, Mem:1Gb

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
```sh
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
    app.run(host="0.0.0.0", port=5001)
```

##### Производительность 1270 RPS 
```sh
Тест wrk -t30 -c300 -d5s http://rec.local:5001/
  6478 requests in 5.10s, 0.87MB read
Requests/sec:   1270.17
Transfer/sec:    174.90KB
```
