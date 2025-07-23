## Cara tes register, login, dll
1. Buka terminal di VsCode

2. Ketik
```bash
py -m pip install flask pymysql
```
```bash
py app.py
```

### Ini curl untuk register
```bash
curl -Method POST http://3.25.114.113:5000/register -Body "nama=Lui&email=lui@mail.com&no_hp=08123456789&password=123456"
```

### Curl login
```bash
curl -Method POST http://3.25.114.113:5000/login -Body "nama=Lui&password=123456"
```

