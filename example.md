# Markdown Demo
---

## External 1.1

```bash
$ git clone https://github.com/kuesy/kubernetes_training
$ cd kubernetes_training
```
---
## External 1.2
- a
- b
- c

---
## External 2
1. nya
2. nyu
3. nyo

---
## External 3.1
```dockerfile
FROM ubuntu
MAINTAINER hoge<fuga@piyo.co.jp>
RUN apt-get update -y && \
apt-get install -y tzdata && \
apt-get install -y apache2
COPY index.html /var/www/html/
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

---
## External 3.2
```kernel
$ git clone https://github.com/kuesy/kubernetes_training
$ cd kubernetes_training
```
---
## External 3.3 (Image)

![External Image](https://s3.amazonaws.com/static.slid.es/logo/v2/slides-symbol-512x512.png)


---
## External 3.4 (Math)

`\[ J(\theta_0,\theta_1) = \sum_{i=0} \]`
---
## External 3.5
foo

{:.fragment}
bar

{:.fragment}
baz
