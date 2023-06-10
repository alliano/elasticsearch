# Apa itu elasticsearch
Elasticsearch merupakan database khusus untuk melakukan pencarian. yang membedakan elasticsearch dengan database lainya yaitu, elasticsearch memiliki fitur-fitur pencarian yang sangat kompleks dan lengkap, sedangkan database-database lainya seperti mysql postgresQl, mongoDb, Sql server dan sebagainya itu untuk fitur pencarianya tidak selengkap elasticsearch.

# Sejarah Elasticsearch
Munculnya elasticsearch itu diawali dengan project yang bernama Apache Lucene. Apache lucene ini dibuat kusus untuk pencarian data dengan performa yang tinggi dan Apache lucene ini merupakan search enggine yang sangant populer. Sayangnya project Apache Lucene ini berbentuk Liberary Java. Jika kita ingin menggunakanya maka kita harus meng embend Apache Lucene ini kedalam project java kita, jadi yang bisa menggunakan Apache Lucene ini hanyalah applikasi yang dibuat dengan bahasa pemograman Java.
![Apache Lucene](https://../images/Apache_lucene.png)

Penggunakan Apache Lucene ini sangatlah kompleks dan lumayan rumit, maka dari itu muculah project yang bernama Compass Project yang mana tujuanya untuk mempermudah penggunakan Apache Lucene. Compass project ini sebenarnya Liberary java juga namum pendekatanya lebih ke ORM(Object Relational Mapping) contohnya seperti Hibbernate kalo di java atau kalo di Php itu Eloquent atau Gorm kalo di bahasa pemograman Golang. Lagi-lagi applikasi selain java tidak dapat menggunakanya karna Compass project ini merupakan ORM nya Apache Lucene dan cara penggunaanya harus di embed kedalam project java. Dan pada akhirnya founder dari Compass project mengakhiri project nya(Compas project) dan membuat project baru yang bernama Elasticsearch.
![Compass project](https://../images/Compass_project.png)

Elasticsearch ini berbeda dengan project-project sebelumnya namun didalam elasticsearch tetap mengembed Apache Lucene(tidak membuat search enggine sendiri) yang membedakan project Elasticsearch dengan project-project lainya yaitu pada project Elasticsearch dibuatkan suebuah system kusus unutuk Database. jadi Berhubung elasticsearch ini system nya adalah database jadi semua applikasi yang dibuat menggunakan bahasa selain java itu bisa menggunakan Elasticsearch, dan juga penggunaan elasticsearch itu menggunakan protokol HTTP.
![Elasticsearch](https://../images/elasticsearch.png)

# Alternatif lain selain Elasticsearch
jika kita tidak inigin menggunakan elasticsearch ada alternativ lain yang bisa digunakan yaitu :
    * Apache Solr
    * Sphink Search
Namun pada seri kali ini kita akan fokus membahas tentang Elasticsearch.

# Kapan Butuh Elasticsearch
Pada umumnya Arsitektur Applikasi itu sebagai berikut..
![arsitektur applikasi](https://../images/Application.png)
dan pada umumnya Applikasi itu biasanya ada fitur untuk pencarian data. Dan biasanya jikalau kita ingin mencari data berdasarkan kolom nama misalnya. Kita bisa menambahkan Index pada kolom nama unutuk mempercepat proses query nya. Dan kita bisa melakukan query sebagai berikut ini :
```SQL
 SELECT * FROM users AS u WHERE u.name = 'Allia';
 ```
 atau jikalau kita ingin mencari data yang diawali kata tertentu dan di akhiri kata tertentu kita juga bisa menambahakan Regex unutk melakukan nya, contohnya seperti berikut ini :
 ```SQL
 SELECT * FROM users AS u WHERE u.name = '%Allia%';
 ```
Pada saat kita menggunakan Regex pada pencarian kita, maka Database tidak akan menggunakan Index untuk melakukan proses pencarianya, melainkan Database akan melakukan pengecekan atau simpel nya Database akan melakukan secaning seluruh data pada kolom nama dan dicocokan dengan kata yang kita input kan. Jikalau data nya itu jumlahnya ribuan bahkan jutaan maka proses pencarianya itu akan sangat lambat.
Atau juga misal ada orang yang mencari data tapi kata-kata nya dipisah atau ada typo dan sebagainya, misalnya :
```SQL
SELECT * FROM users AS u WHERE u.name = '%Al liya%';
```
tentusaja ini tidak akan mendapatkan data yang diinginkan.

## Agregate
Misalnya kita ingin menampilkan data produk dan ingin mengagregate beberapa fieldnya contohnya kita ingin mengagregate berdasarkan Lokasi
![Lokasi](https://../images/Lokasi.png)
Pengiriman dan Promo
![](https://../images/Compass_project.png)
Jikalu kita menggunakan database bisa(postgreSQL, Mysql, mongoDb, dan sebagainya) sebenarnya lumayan gampang kita tinggal menggunakan fitur group by saja. namun bagaimana jika kita ingin mengagregate dalam 5 kategori tentusaja yang akan kita lakukan pertama yaitu mengquery datanya lalu mengrouping sebanyak 5x, jadi totalnya 6 proses eksekusi, dan pastinya akan lambat.

## Kompleks search Data
Atau misalnya kita ingin fitur auto complelete pada query, atau kita ingin menambahakan pencarian berdasarkan singkatan misalnya Jabar, Jatim, JKT dan sebagainya. Tentusaja itu sangatlah ribet jikalau kita menggunakan Database biasa.

# Arsitektur Setelah Menggunakan Elasticsearch
Setelah kita menggunakan elasticsearch maka arsitektur dari applikasi yang kita buat akan sedikit berbeda. karena kita akan menggunkan dua database :
  * Database untuk penyimpanan(Mysql, PostgreSQL, atau mongodb dan sebagainya)
  * Database untuk pencarian(Elasticsearch)
Gambaranya dapat dilihat sebagai berikut :
![app arc](https://../images/Application_architecture.png)

# Arsitektur penggunaan Elasticsearch
Saat kita menjalankan elasticsearch, itu bisa kita sebut kita menajalankan node elasticsearch. Pada real case(di production) kita tidak akan menjalalankan 1 node elasticsearch, biasanya kita akan menjalankan lebih dari 1 atau 3 node elasticsearch, perlu diingat saat kita menjalankan node elasticsearch direkomendasikan jumlah node nya itu ganjil, karena unutk mengatisipasi jikalau terjadi splitbranch. ---
Saat semua node kita terkoneksi maka gambaranya sebagai berukut :
![node_connected](https://../images/Nodes_connected.png)
Saat semua node terkoneksi maka kekonsistenan data akan selalu terjaga(terhindar dari redundancy). Namun saat hubungan network antar node kita terputus(terjadi split branch)
![node_notconnect](https://../images/node_notConected.png)
maka node elasticsearch yang akan digunakan adalah node yang paling banyak terhubung dengan node lain. dalam gambar diatas adalh node3, node4, dan node5. ---
node1 dan node2 tidak akan digunakan samapi network kemabali terhubung(split branch berakhir). ---

jikalu kita menjalankan node elasticsearch dengan jumlah genap maka yang akan terjadi jikalau terjadi split branch dan jumlah kedua sisi node itu sama 
![](https://../images/node_balance.png)
maka elasticsearch akan menjalankan kedua sisi node yang terputus. karena terjadi split branch dan kedua sisi node berjalan maka yang akan terjadi adalah data tidak lagi konsisten(redundancy) dan saat node nya terhubung kembali maka akan terjadi ERROR karena datanya tidak konsisten. maka dari itu disarankan saat kita menjalankan node elasticsearch harus ganjil.

# Elasticsearch cluster
Saat kita menjalankan beberapa node dan beberapa node tersebut saling terhubung maka kita sebut dengan cluster(kumpulan dari bebrapa node)
![cluster_elastricsearch](https://../images/elasticsearch_cluster.png)
Didalam satu cluster semua node elasticsearch akan saling terhubung dan bekerja sama.

## Masterless
Tiap node elasticsearch itu berjalan dengan konsep masterless. Artinya dalam cluster elasticsearch itu tidak ada master nya atau tidak ada database utama nya jadi misal kita melakukan operasi penyimpanan pada node1 bisa saja datanya di simpan di node 3 dan sebagainya, atau saat kita query di node3 misalnya dan data yang kita cari itu adanya di node 1 maka node3 akan meneruskan query nya ke node1 dan menegembalikan data yang dicari.

# Database
Diadalam elasticsearch itu tidak ada yang namanya Database namun yang ada hanyalah Index. ---
Index ini dapat kita analogikan sebagai Tabel di dalam database.
Lantas gimna jika kita ingin menggunkan elasticsearch untuk menyimpan beberapa database ? ---
untuk menatasi hal tersebut kita bisa menggunakan prefix saat penamaan Index nya, misalnya :
 * payment_user
kata yang didepanya sebagai prefix untuk menandai bahwa Index payment_user itu milik database payment.

# Index
Saat kita membuat index pada elasticsearch, index tersebut akan disimpan kedalam Lucene document. jadi saat kita menambahakan data pada index maka data tersebut akan disimpan didalam Lucene document. 
![index](https://../images/index.png)
Didalam Lucene document itu sebenarnya isinya cuman atribut Key dan value, contohnya :
| Atributes   | vaue              |
|*************|*******************|
| Name        | Allia             |
| Last_name   | Azahra            |
| Email       | azahra@gmail.com  |
| ...         | ...               |

# Sharding
Sharding itu adalah memotong Index yang kita buat menjadi bebrapa partisi/bagian yang di distribusikan atau di sebarkan ke semua node yang ada didlam culuster elasticsearch. Secara default saat kita membuat Index, elasticsearch akan membagi Index tersebut menjadi 5 partisi.
![partition](https://.../../images/partition.jpg)
Namun jikalau kita ingin mengubah setingan default nya, kita bisa mengaturnya saat kita membuat Index.
Seperti yang terlah dikatakan diatas potongan dari index akan di distribusikan ke beberapa node yang ada didalam elasticsearch cluster.
![node_elastic](https://../images/shards_nodes.jpg)
Pendistribusian patisi ke node-node itu dilakukan secara otomatis oleh elasticsearch, dan penempatan partisinya random.

# Keuntungan Sharding
Keuntungan dari sharding(memecah index menjadi beberapa bagian) dapat membuat node-node tidak keberatan karena pada saat kita melakukan operasi pada elasticsearch maka semua node akan bekerja sama. Kenapa bisa bekerjasama ?? karena Index nya telah terdistribusi ke semua node. ---

# Replika
Repikasi merupakan teknik unutk menduplikasi index pada elasticsearch. By default saat kita membuat index, elasticsearch juga akan membuat 1 replika unutuk index tersebut. Dan data replikasi tersebut tidak akan di simpan pada node yang sama, misalnya kita membuat index dan index tersebut disimpan pada node1 maka replikanya bisa saja ada pada node2 atau node3 dan seterusnya.
![image](https://../images/replica.jpg)
Namun saat kita menambahkan node baru dalam cluster kita nanti, maka elasticsearch akan melakukan rebelance jumlah replika dan partisinya, maksudnya replika dan primary akan di ditribusikan pada node yang baru.
![replica_rebelance](https://../images/replica_rebelance.jpg)
dengan begitu data akan selalu available walapun nanti terjadi salah sattu node mati.

# Document Routing
Pada saat kita akan menyimpan document, maka elasticsearch akan menentukan document tersebut itu akan di simpan pada shard keberapa. Cara penyimpanan document tersebut tidak dilakukan secara random, melainkan menggunakan cara routing. elasticsearch akan melakukan routing dengan cara hashing dengan menggunakan algoritma murmurhash, jadi elasticsearch akan melakukan konversi kedalam bentuk binary dengan menggunakan algoritma murmurhash pada unique id dari request yaang dikirim atau secara otomatis di generatekan oleh elasticsearch, lalu hasil hashing nya itu akan di modulus dengan jumlah shareds atau partisi yang ada.
rumus hashing nya bisa dilihat sebagai berikut : 
```
shard = hash(unique_id) % jumlah shared/partisi
```
routing tersebut berlaku untuk menyimpan data ataupun mengambil data.

# Distribusi Documents
Saat kita melakukan proses Create, Indexing, Dellete yang terjadi pada elasticsearch itu sebagai berikut :
![distributed_document](https://../images/dictribuded_documents.jpg)
jadi tahap pertama itu elasticsearch akan melakukan routing setelah itu maka elasticsearch akan mengetahui data tersebut disimpan di partisi yang mana, setelah elasticsearch mendapatkan data tersebut pada partisinya maka elasticsearch akan melakukan operasi(Create, Indexing, Delete) pada shards/partisi primary, setelah berhasil melakukan operasi tersebut maka request tersebut akan dipropagasikan secara palarel kepada replika-replika nya yang berada di node-node pada cluster elasticsearch.

# Retriving Documents
Untuk proses select atau retrive atau get documents(Data) itu sedikit berbeda dengan proses Create, Indexing dan Delete ---
untuk proses retrive itu sebagai berikut :
![retriving_documents](https://../images/retriving_docs.jpg)
Jadi proses pertama elasticsearch akan melakukan routing terlebih dahulu, setela proses routig selesai maka elasticsearch mengetahui document tersebut berada pada partisi yang mana, pada saat mengambil document nya elasticsearch akan mengambil document tersebut secara random dari partisi primary atau replikasi nya, setelah berhasil mengambil maka data tersebut akan dikemabalikan ke node yang menerima request tersebut dan node tersebut akan memberikan document tersebut melalui response.

# Updating Documents
Untuk proses Update ini mirip seperti dengan proses Create, indexing, Delete. bisa dilihat pada diaram berikut ini :
![update_document]()
Jadi pertama yang dilakukan yaitu routing terlebih dahulu unutuk mengetahui data tersebut berada pada shared yang mana. di contoh tersebut misalnya setelah dilakukan routing ternyata datanya berada pada shard ke 0, maka elasticsearch akan melakukan proses update nya kepada shard 0 tersebut, setelah proses update selesai maka request tersebut akan di propagasi ke repika-replika nya palarel yang berada pada node-node didalam cluster elasticsearch.

# Immutable Document
Elasticsearch menyimpan dokument nya itu secara immutabe didalam storage komputer(Hardisk/SSD). Artinya document tersebut tidak akan pernah berubah bentuknya sedikit pun. Keuntungan Immutabele yaitu :
  * tidak perlu ada proses locking data sebelum melakukan operasi.
  * tidak perlu kawatir dengan proses yang palarel ketika mengakses atau mengubah data.
  * dan sebagainya.

# Update dan Delete
Berhubung documnet yang disimpan pada elasticsearch itu bersibat immutable, maka document tersebut tidak bisa diubah ataupun di hapus. ---
jadi untuk proses delete sebenarnya itu elasticsearch akan membuat file yang ber extensi .del dan akan memasukan dokument yang di delete tersebut kedalam file .del, simpelnya documnet yang di hapus tersebut hanyalah di mark as delete pada dokument .del ---
untuk proses update yang terjadi yaitu, documnet tersebut di mark as delete didalam file .del, dan elasticsearch akan melakukan proses insert data yang baru. ---
namun jagan kawatir jikalau data akan membengkak. elasticsearch memiliki scheduler yang digunakan untuk menghapus data tersebut secara real pada kurun waktu tertentu.

# Distirbuted Search
Disribusi search ini merupakan tahapan yang lumayan kompleks dibandingkan proses Isert, Update, Delete, Retrive. Proses search menjadi kompleks karena elasticsearch akan melakukan search kepada semua shards/partisi yang ada didalam elasticsearch cluster.
proses searchig pada elasticsearch ini dibagi menjadi 2 fase :
* Query phase
![Query_pahse]()
Misalnya request masuk dari node-3, setelah itu elaticsearch akan mencari data tersebut kesemua shards/partisi, pada proses ini elasticsearch tidak memprioritaskan pencarian pada shards/partisi primary melainkan elasticsearch akan mencari pada semua partisi(Replica, Primary) secara Palarel dan random. ---
Setelah itu elasticsearch akan mengembalikan id document yang didapat pada shards/partisi pada saat proses searching dan mengembalikan id tersebut ke node-3 ---
Perlu diingat jikalau request tersebut itu masuk dari node-3, maka elasticsearch tidak akan melakukan searching pada node tersebut melainkan elasticsearch akan mempropagasi requst tersebut kepada node-node lain, misalnya pada kasus ini adalah node-2 dan node-3
* Fetch Phase
![Fetch_phase]()
Untuk alur dari Fetch phase ini sama denga alur Query pahase. Setelah selesai melakukan Query phase maka node-3 akan mendapatkan id yang match dengan key pencarian, seteh itu sebelum elasticsearch masuk dalam fetch phase, elasticsearch akan mensorting id tersebut, misalnya elasticsearch mensorting 100 id documents pertama. Setelah itu proses fetching dilakukan oleh elaticsearch secara palarel dan belance ke shards/partisi, setelah itu hasil pencarian akan dikembalikan ke node-3 dan node-3 akan mengirimkan hasil pencarian tersebut melalui http response.

# Deep pagination
jikalau kita memiliki 5 shards dalam cluster dan kita melakukan proses searching dengan limit hasil 100 data, maka tiap shard/partisi akan mengemabalikan 100 data, artinya kita akan mendapatkan 500 data yang harus diproses. ---
jikalau terlalu banyak data yaang harus diproses maka proses shorting sebelum melakukan fetching biasanya akan membutuhkan resource badwith dan CPU yang besar. ---
Maka dari itu unutk melakukan paging yang sagat dalam tidak disarankan. ---
Jikalau kita igin mengambil semua data document yang ada pada elasticsearch maka kita jangan menggunakan searching melakinkan menggunakan fitur yang namanya Scroll. Scroll ini adalah teknik scaning semua data yang ada pada elasticsearch cluster.

# Installation
Untuk proses instalasinya, tahap pertama kita bisa download elasticsearch dari link berikut ini :
![elasticsearch_download](https://www.elastic.co/downloads/elasticsearch) ---
dan jagan lupa disesuaikan dengan versi device temn2(Windows, Mac, Linux) ---
Setelah proses download selesai, tehap selanjutnya kita extrac file elasticsearch nya, setelh selesai extract kita akan mendapatkan file sebagai berikut :
![base_file_elasticsearc](https://../images/base_file_elasticsearch.png) ---
folder bin merupakan folder yang berisi command-command yang digunakan untuk menjalankan elasticsearchnya ---
folder config merupakan folder yang berisi konfigurasi elasticsearch dan konfigurasi JVM(Java Virtual Machine) --
kita akan fokus kepada kedua folder tersebut.

# Configuration Elasticsearch
Tahap selanjutnya sebelum kita menjalankan node elasticsearch, kita harus mengkonfigurasi elasticsearch nya terlebih dahulu, berikut ini merupakan konfigurasi dasar dari elasticsearch :
``` yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 
# ini unutk menamai clusternya
cluster.name: elasticsearch-cluster
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
# untuk nama node nya
node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
# ini merupakan path yang akan kita gunakan unutk menyimpan data elasticserch
#path.data: /path/to/data
#
# Path to log files:
#
# ini merupakan path yang akan kita gunakan unutk menyimpan log elasticserch
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
#network.host: 192.168.0.1
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
# 
# ini untuk meng settion port yang akan digunakan
http.port: 9205
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
# discovery.seed_hosts: ["127.0.0.1"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
# cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false
# ---------------------------------- Security -----------------------------------
xpack:
  security:
    enabled: false
```
# Configuration JVM
Tahap selanjutnya setelah mengkonfigurasi elasticsearch, kita juga harus mengkonfigurasi JVM nya, terlebih kususnya pada bagian penggunaan minimum dan maksimum storage yang digunakan. berikut ini merupakan konfigurasi dasar nya :
```yaml
################################################################
##
## JVM configuration
##
################################################################
##
## WARNING: DO NOT EDIT THIS FILE. If you want to override the
## JVM options in this file, or set any additional options, you
## should create one or more files in the jvm.options.d
## directory containing your adjustments.
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/8.7/jvm-options.html
## for more information.
##
################################################################



################################################################
## IMPORTANT: JVM heap size
################################################################
##
## The heap size is automatically configured by Elasticsearch
## based on the available memory in your system and the roles
## each node is configured to fulfill. If specifying heap is
## required, it should be done through a file in jvm.options.d,
## which should be named with .options suffix, and the min and
## max should be set to the same value. For example, to set the
## heap to 4 GB, create a new file in the jvm.options.d
## directory containing these lines:
##
## -Xms4g
## -Xmx4g
## Hal pertama yang harus diperhatikan adalah :
## -Xms4g dan Xmx4g
## dua property tersebut merupakan settingan minimu maksimum penggunaan memory pada elasticsearch
## untuk nilai nya direkomendasikan sama, misalnya jika minimum memory usage nya itu 1gb maka maksimum
## nya juga 1gb dengan tujuan agar tidak membebani getbage collector java nya.
-Xms1g
-Xmx1g
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/8.7/heap-size.html
## for more information
##
################################################################


################################################################
## Expert settings
################################################################
##
## All settings below here are considered expert settings. Do
## not adjust them unless you understand what you are doing. Do
## not edit them in this file; instead, create a new file in the
## jvm.options.d directory containing your adjustments.
##
################################################################

-XX:+UseG1GC

## JVM temporary directory
-Djava.io.tmpdir=${ES_TMPDIR}

## heap dumps

# generate a heap dump when an allocation from the Java heap fails; heap dumps
# are created in the working directory of the JVM unless an alternative path is
# specified
-XX:+HeapDumpOnOutOfMemoryError

# exit right after heap dump on out of memory error
-XX:+ExitOnOutOfMemoryError

# specify an alternative path for heap dumps; ensure the directory exists and
# has sufficient space
-XX:HeapDumpPath=data

# specify an alternative path for JVM fatal error logs
-XX:ErrorFile=logs/hs_err_pid%p.log

## GC logging
-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,level,pid,tags:filecount=32,filesize=64m
```
# Start server Elasticsearch

Setelah selesai mengkonigurasi elasticsearch dan JVM nya, kita bisa menjalankan node elasticsearch dengan command sebagai berikut :
``` bash
./bin/elasticsearch
```
Untuk memastika elasticsearch sudah runnign kita bisa mengecek melalui blowser dengan cara kita isikan alamat loacalhost dan port yang telah kita setting pada elasticsearch.yaml. misalnya localhost:9201, jikalau elasticsearch berhasil dijalankan maka akan muncul halaman.


# Setup Cluster
Sebelum kita mengseting cluster elasticsearch pastikan kita telah memiliki beberapa node elasticsearch.
Untuk setup cluster nya yang perlu diperhatikan yatiu konfigurasi discovry pada konfigurasi elasticsearch.yaml nya. Berikut ini merupakan konfigurasi dasar nya :
``` yaml
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]

# ini digunakan unutk membentuk cluster nya, maksudnya elasticsearch akan menentukan daftar host
# yang digunakan untuk pembentukan cluster saat elasticsearch dijalankan. Berhubung kita menggunakan
# lokal komputer, kita bisa set ip 127.0.0.1 atau localhost
discovery.seed_hosts: ["127.0.0.1"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
# ini digunakan untuk menginisiasi master node saat pertama kali cluster elasticsearch nyala, maksudnya
# elasticsearch akan memilih salahsatu dari node-node yang kita registrasikan namanya sebagai master node
# rekomendasi untuk initial_master_node ini kita meregistrasikan jumlah node nya ganjil misal 3, 5 dan 
# seterusnya.hal tersebut dilakukan untuk menghindari split_branch network.
# jika kita registrasikan tiga node pada initial_cluster_nodes nya maka elasticsearch cluster dan node nya 
# akan bisa digunakan jikalau node yang nyala itu setengah dari jumlah node yang kita registrasikan atau 
# lebih. Jikalau yang nyala itu hanya 1 node maka elasticsearch tidak akan mau bekerja, minimal setengah
# jumlah node yang kita registrasikan pada initial_master_nodes nya, dalam kasus ini kita meregistrasikan 
# 3 node maka minimal yang harus nyala 2 node agar elasticsearch mau bekerja.
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false
```
Stelah selesai mengkonfigurasi cluster nya, kita bisa jalankan elasticsearch nya.

# Create Index
Langkah pertama saat kita akan menggunakan elasticsearch yaitu membuat Index. ---
Index disini digunakan sebagai media penyimpanan data. Jikalau kita asumsikan sebagai RDBMS index ini seperti tabel. Untuk membuat index kita membutuhkan HTTP Client(Postman atau Insomnia dan applikasi sejenisnya). ---
cara untuk membuat index yaitu dengan mengirim requst sebagai berikut :
``` Http
http://{host}:{port}/{nama_Index}
```
misal nya jikalau kita ingin membuat index users : http://localhost:9201/users
![create_index](https://../images/create-index.png)

# Index name Limitations
Ada beberapa batasan saat kita membuat nama index, yaitu :
* Tidak boleh menggunakan huruf kapital
* Tidak boleh menggandung /,\,*,?,",<,>,|, ``, (special character), #
* Tidak boleh mengandung (:)
* Tidak boleh dimulai dengan karakter -, +, _

Referensi :[create index requirement
](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html#indices-create-api-path-params)

# Index Setings
Saat kita membuat index kadang kita ingin menetukan berapa replika dan shard/partisi index tersebut. ---
Untuk menseting hal tersebut, kita bisa melakukanya dengan cara mengirim requst body berupa json sebagai berikut : 
``` json
PUT /users
{
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 2 
    }
  }
}
```
* peroperti number_of_shards merupakan properti untuk menentukan berapa partisi yang digunakan untuk Index tersebut.
* number_of_replicas merupakan properti untuk menentukan berapa replika yang dimiliki index tersebut. ---

example :
![create_index_with_setings](https://../images/create_index_with_settings.png)
Untuk megetahui lebih detail index tersebut, biga kirimkan requst dengan menggunakan Http method get dan diikuti dengan nama index nya, contoh :
``` bash
http://localhost:9201/users
```
# perform Create, Update, Delete
Operasi Create, Update, Delete dalam elasticsearch biasa disebut dengan Indexing.

## Create
Untuk melakukan create kita bisa menggirimkan requst body dengan HTTP method PUT dan diikuti dengan data json. contohnya :
PUT http://localhost:9201/users/_doc/unique_id
``` json
{
  "name" : "Alliano",
  "age" : 20
  "wife" : {
    "name" : "Allia Azahra",
    "age" : 19
  }
}
```
_doc merupakan tipe data yang kita akan gunakan untuk menyimpan data pada index. ---
disarankan pada 1 index gunakan 1 tipe data penyimpanan pada saja, misalnya jikalau _doc maka saat kita create data baru maka tipe nya _doc juga.

## Update
Untuk proses update pada elasticseac itu sama seperti proses Insert, hanya saja pada saat kita mengupdate kita juga harus mengirimkan unique id dari data yang ingin kita update, jikalau unique id tersebut tidak ada pada elasticsearch maka proses tersebut akan menjadi proses insert. contoh :
PUT http://localhost:9201/users/_doc/1
``` json
 "name" : "Ali",
  "age" : 20
  "wife" : {
    "name" : "Allia Azahra",
    "age" : 19
  }
```
perlu diperhatikan pada saat proses update data yang harus dikirimkan harus full payload, maksudnya data yang tidak di update juga harus dikirimkan. Jika kita tidak mengirimkanya maka data tersebut akan hilang.

## Delete
Pada proses delete ini, cara melakukanya itu sama seperti cara create dan update yang membedakan hanyalah HTTP method nya saja. untuk operasi delete kita menggunakan HTTP method Delete. Contoh :
DELETE http://lcalhost:9201/users/_doc/1

# Search Query
Untuk melakukan search itu cukuplah mudah, kita hanya perlu mengirimkan beberapa data json sebagai berikut ini :
POST http://{host}:{port}/{index}/_search
``` JSON
{
  "query" : {
    "match" : {
      "{key}" : "{keyword}"
    }
  }
}
```
Untuk attribut query itu wajib dikirimkan, dan atribut match itu juga wajib kita kirimkan untuk mencocokan keyword pencarian dengan dokumen yang ada di dalam elasticsearch. setelah atribut match lalu diikuti dengan nama key pencarian, misalnya kita ingin mencari dokumen dengan key name dengan nama Allia Azahra.
``` JSON
{
  "query" : {
    "match" : {
      "name" : "Allia Azahra"
    }
  }
}
```

# Bool Query
Bool query ini merupakan tipe query yang digunakan untuk melakukan pencarian yang lebih kompleks dan fleksibel. Bool query ini memmungkinkan seorang programmer untuk menggabungkan beberapa kondisi pencarian menggunakan operator logika seperti AND, OR, dan NOT.
cotoh beberapa operasi query bool :
* must, bool query ini seperti operator logika AND
* must_not, bool query ini seperti operator logika NOT
* should, bool query ini sama seperti operator logika OR
  
refrensi : [bool_query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)

# Bool query must
bool query must merupakan query yang perilakunya seperti operator logika, query ini biasa digunakan unutuk mencocokan bebrapa key dalam document, dan jikalau satu key tidak memenuhi kondisi yang telah kita tentukan maka query ini akan diaanggap gagal. ---
Contoh kita ingin mencari dokumen dengan key first_name Allia, last_name Azahra, address Bandung. maka maka query bool nya, yaitu :
``` JSON
{
  "query" : {
  "bool" : {
    "must" : [
      {
        "match" : {
          "first_name" : "Allia"
        }
      },
      {
        "match" : {
          "last_name" : "Azahra"
        }
      },
      {
        "match" : {
          // misalnya attribut address ini merupakan suatu object yang menyimpadan data alamat
          // salahsatu nya data provinsi maka ktia bisa lakukan sebagai berikut
          "address.province" : "Bandung"
        }
      }
    ]
    }
  }
}
```
# Bool query must_not
bool query must_not merupakan query yang perilakuknya seperti operator logika NOT, query ini biasanya digunakan untuk pengecualian pencarian. misal nya kita ingin mencari user yang memiliki hobi selain Reading, maka kita bisa membuat query nya :
``` JSON
{
  "query" : {
    "bool" : {
      "must_not" : {
        "match" : {
          "hobbies" : "Reading"
        }
      }
    }
  }
}
```
# Bool query should
boll query should merupakan query yang memiliki perilaku seperti operator logika OR, query ini biasanya digunakan untuk mencari data dalam suatu dokumn dengan bebrapa key namun jikalau salah satu key tidak terpenuhi query tersebut tetap dianggap berhasil. contohnya kita ingin mencari address dari user yang provinsinya di Maluku atau Bandung, jadi user yang profinsinya di bandung ataupun di Maluku, maka akan terseleksi.
``` JSON
{
  "query" : {
    "bool" : {
      "should" : {
        "match" : [
          {
            "address.province" : "Bandung"
          },
          {
            "address.province" : "Maluku"
          }
        ]
      }
    }
  }
}
```
# Agregation query
Aggregate pada Elasticsearch itu terbagi menjadi tiga aspek, yaitu :
* Bucket aggregations ---
Bucket aggregations digunakan untuk mengelompokkan data dalam kategori atau segmen yang berbeda, sehingga memungkinkan analisis dan visualisasi yang lebih terperinci terhadap data.
* Metrics aggregations ---
Aggregate Metrics pada Elasticsearch merujuk pada metode untuk mengumpulkan, menggabungkan, dan menganalisis data agregat secara statistik. Dalam konteks Elasticsearch, aggregate metrics digunakan untuk melakukan operasi penghitungan atau perhitungan statistik pada data yang ada dalam indeks Elasticsearch.
* Pipline aggregations
Aggregate pipeline dapat digunakan untuk menggabungkan beberapa aggregasi, memfilter hasil agregasi, memodifikasi hasil agregasi, dan melakukan operasi matematika atau statistik pada hasil agregasi. 

Contoh penggunaan agregate :
``` JSON
{
  "aggs" : {
    "{agregate_name}" : {
      "terms" : {
        "field" : "{field_name.keyword}"
      }
    }
  }
}
```