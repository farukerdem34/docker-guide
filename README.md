<div align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/7/79/Docker_%28container_engine%29_logo.png" />
</div>

# Neden Docker?
- Taşınabilir
- Optimize Kaynak Tüketimi
- Hızlı Çalıştırılabilir (CI/CD)
- [İzole](#sandbox-nedir)
- [Ölçeklendirilebilir](#microservis)
## Linux Paketleri Nasıl Çalışıyor?
GNU/Linux Windows'a göre bağımlılıklarını (paketlerini) biraz farklı ve verimli bir yolla saklamayı tercih etmekte. Windows bu noktada depolamadan feragat ederek her uygulama kurulumunda uygulamanın tüm bağımlılıklarını kendisiyle beraber getirerek kurulumda hiç yokmuş gibi kurmasını ister.

Fakat GNU/Linux bağımlılıkları `/usr/lib/`, `/usr/lib64`, `/usr/libexec` gibi dosya yollarında bir program o bağımlılığa ihtiyaç duyduğunda sisteme yükler ve başka bir program bu bağımlılığa ihtiyaç duyacağı zaman tekrardan bağımlılığı yüklemeyerek onu `link`ler ve tüm programlara bu bağımlılığı kardeş kardeş kullanmalarını söyler.

GNU/Linux'un bu yaptığı depolama ve her program kurulumunda beklenen süre gibi sorunları yüksek verimlilikte yok etmektedir. Yeni kurulmuş bir GNU/Linux işletim sisteminin 1-2 GB depolama harcamasının arkasındaki en büyük neden budur.

Fakat her gülün dikeni var ve `bağımlılık cehennemi` de GNU/Linux mecrasında bunlardan bir tanesi. Her ne kadar sık rastlanılmayan bir durum olsa da karşılaşıldığında ciddi sorunlara yol açmaktadır fakat neden? Debian tabanlı işletim sistemlerinde `deb`, RHEL/Fedora tabanlı işletim sistemlerinde `rpm` paketleri kullanılmakta ve bu paketler sistem yöneticileri tarafından çok büyük ihtimalle Debian tabanlı işletim sistemlerinde `apt`, RHEL/Fedora tabanlı işletim sistemlerinde `yum` ya da `dnf` paket yükleyicisi araçları kullanılarak yapılmakta. Bu paket yöneticileri bir bağımlılığın (paketin) yüklenmesi, silinmesi, güncellenmesi gibi sistem uygulamalarının çalışması için yapılması gereken her şeyi yapmakta. Ama bazen bazı uygulamaların aynı paketin farklı sürümlerine ihtiyaç duyduğu durumlarda bu paket yöneticileri işin içinden çıkamayarak `bağımlılık cehennemine` düşebiliyorlar.

İşte burada [`sandbox`](#sandox) yardımımıza koşuyor.
## Sandbox
Kelimenin tam anlamıyla bir kum havuzu hayal edin ve bütün uygulamaların kendi kum havuzunda oynamakta zorunda olduğunu düşünün, tek bir sunucuda hem `Nginx 2.0` hem `Nginx 3.0` kullanabiliyorsunuz ve ikisinin birbirine hiçbir zararı yok.

Evet `bağımlılık cehenneminden` kurtulduk, bu kadar mı? Değil. Ama devamı için [container](#container) diyelim.
### Container
Konteynerleri yük gemisindeki konteynerler gibi düşünebilirsiniz, nasıl ki bir konteynere kendi içinde herhangi bir zarar geldiğinde yanındaki konteynerin ruhu duymuyorsa `Docker` da işleri aynen böyle yürütüyor. Geliştirdiğiniz Web uygulamasında güvenlik zaafiyeti bulundu ve sunucunuza zarar mı verildi? Web sunucumuz `Nginx`'in ve veri tabanınız `MySQL`in bunu pek umursayacağını sanmıyorum, Web uygulama konteynerinizin içinde ciddi sorunlar olabilir ama o onların sorunu.

Konteynerlerinze `--priviliged` yetkisini vermediğiniz sürece - çoğunlukla bu yetkiyi yalnızca güvenlik duvarları, IDS vs IPS gibi sistemlere vermek zorunda kalırsınız, konteyneriniz kendi yağında kavrulmaya devam edecektir.
## Microservis
Web uygulamanızın web sunucusu `Nginx` ve veri tabanı `MySQL`den bahsetmiştik, bu üç konteyner birer `Microservice` adından da anlaşılacağı üzere, micro. Bu servislerin içine bir tane de `Nginx` Reverse Proxy kurarak 4 `Microservice` kullanabilirsiniz. 2 `Nginx`, 1 Web uygulaması, 1 `MySQL`, hatta bir tane de `Certbot` kurup `SSL` [sertifikalarını da ona yaptırabilirisiniz](https://forum.manjaro.org/t/mirrors-download-aur-manjaro-org-ssl-certificate-expired/115074).

Bu servisler sizin makinenizde gerçekten çalıştığında başka makinelerde de çalışacaktır, `Docker` güvencesi ;)

# Docker
[Bu adresten](https://www.docker.com) resmi docker dokümantasyonuna bakarak kurulum yapabilirsiniz veya aşağıdaki komutu komut satırında çalıştırarak en güvenilir kurulumu yapabilirsiniz.
```bash
curl https://get.docker.com | bash
```
talimatları izledikten sonra `Docker` sisteminize kurulacaktır. `Docker` sisteminize kurulduktan sonra `docker` komutunu her kullanmanız gerektiğinde `sudo` komutunu da başına eklemeniz gerekmekte ve bu can sıkıcı olabilir. `Production` sunucusunda bunu yapmamak kaydıyla **şimdilik** kullanıcınızı `docker` grubuna ekleyerek bundan kurtulabilirsiniz.
```bash
sudo usermod -aG docker $USER
```
ardından aşağıdaki komutu çalıştırarak kullanıcınızın dahil olduğu grupları güncelleyerek yetkilerini kullanabilirsiniz.
```bash
newgrp docker
```
## Nginx Çalıştıralım!
O kadar lafını ettik, çalıştırmadan olmaz.
```bash
docker run -it -d --name my-nginx -p 80:80 nginx:latest
```
çalıştırmadan önce bu parametrelere bi bakalım.

- `-it` komutun gerçek bir terminalde çalışıyormuş gibi davranmasını sağlar
- `-d` containerın terminale bağlı kalmadan arkaplanda özgürce çalışmasını sağlar
- `--name` adından da anlaşılacağı üzere containera isim koyar
- `:latest` hangi sürümü istediğimizi belirtir
- `-p` işte en sık kullanacağınız parametrelerden birisi, bu komutta container içindeki `80` portunu ana makinemizdeki `80` portuna yönlendirmekte, atamakta. Eğer `-p 8080:80` olarak girilmiş olsaydı container içindeki `80` portunu ana makinenin `8080` portuna yönlendirecek, atayacaktı.
şimdi çalıştıralım! `http://localhost` veya `http://127.0.0.1` adresine gidelim.

![nginx fotoğrafı](https://i.hizliresim.com/irrtiif.jpg)

### Container ile Etkileşime Geçelim

`docker container ls` komutuyla çalışan tüm containerlarınızı görebilirsiniz, bunun için `docker ps` komutunu da tercih edebilirsiniz. `docker container ls -a` komutuyla ise çalışan çalışmayan tüm containerları görüntüleyebilirisiniz.

`docker container stop <container-id>` komutuyla container'ı durdurabilir, `docker container start <container-id>` komutuyla containerınızı tekrar başlatabilirisiniz, bunun için container adını da kullanabilirsiniz.

`docker logs <container-id>` komutuyla container loglarını okuyabilir, bu komut `-f` parametresini ekleyerek tam zamanlı olarak yazılan tüm logları görüntüleyebilirsiniz.

#### [Dockerhub](https://hub.docker.com/)
Dockerhub sayısız konteyner bulabileceğiniz bir platformdur, çalıştırdığımız `Nginx` yansısı buradan geldi ve fazlası da var. Örneğin `nginx:latest` yerine `nginx:alpine` kullanarak `Nginx`'in [`Alpine Linux`](https://alpinelinux.org/) kullanarak oluşturulmuş yansısını kullanabilirsiniz. [`Alpine Linux`](https://alpinelinux.org/) konteyner dünyasının özel Linux dağıtımlarından birisidir.

O zaman `Nginx`'in şu [`Alpine Linux`](https://alpinelinux.org/) versiyonunu da yükleyelim
```bash
docker pull nginx:alpine
```

`docker images` veya `docker image ls` komutunu kullanarak yüklenen yansıları listeleyebilir, `docker rmi <yansı-idsi>` veya  `docker image rm <yansı-idsi>` komutunu kullanarak yansıları silebilir, `docker image prune` komutuyla da hiçbir konteynere bağlı olmayan yansıları kaldırabilirsiniz.

Bu konteynerlerin kendilerine ait sanal depolama diskleri (Volume), sanal `Docker` ağları (Docker Network) da bulunmakta. Evet, `Docker` kendi içinde ağ yapılanmasına da olanak sağlamakta. Bi' inceleyelim konteynerimiz ne durumda?

```bash
docker inspect my-nginx
```
oldukça fazla ve değerli bilgiler elde ediyoruz, mesela hangi `Docker ağında` ve hangi `IP` adresine sahip, hangi portları hangi porta yönlendiriliyor.

`docker volume ls` komutuyla şu an kadar oluşturduğunuz konteynerlerin depolama birimlerini listeleyebilirisiniz, insan dilinden biraz uzak hash değerleri görüyor olabilirsiniz. Bundan kurtulmak için konteyneri oluşturma aşamasında `-v` parametresini kullanarak konteyner içerisinde hangi dosya(ları) `volume`e ilave etmek istediğinizi belirtebilirsiniz.
```bash
docker run -it -p 80:80 --name my-nginx -v nginx-volume:/etc/nginx nginx:latest
```

`docker inspect nginx-volume` komutunu kullandığımızda `my-nginx` konteynerinin içerisindeki `/etc/nginx` yolunun dahil edildiği sanal konteyner depolama biriminin özelliklerini ve ana makinede nerede saklandığını öğrenebilirsiniz.

### Docker Ağları

```bash
docker network <parametre>
```
komutunu kullanarak `Docker` ağı işlemlerini yapabilirsiniz.

```
docker network create --attachable --subnet 10.10.10.0/24 my-network
```

- `--attachable` konteynerleri sonradan bu ağa dahil edebilmemizi sağlamakta
- `--subnet` ağ subnet ayarını belirlememizi sağlamakta

```bash
docker run -it -p 80:80 --name my-nginx -v nginx-volume:/etc/nginx --network my-network --ip 10.10.10.10 nginx:latest
```

oluşturduğumuz konteynerin `my-network` ağına dahil olup opsiyonel olarak belirtilen `--ip` parametresiyle manuel olarak `IP` ataması yapmamızı sağlamakta.

# Docker Compose
