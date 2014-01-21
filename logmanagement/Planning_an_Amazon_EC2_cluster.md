Planning an Amazon EC2 cluster

Amazon EC2 cluster でのプラン

DataStax provides an Amazon Machine Image (AMI) to allow you to quickly deploy a multi-node Cassandra cluster on Amazon EC2.

`DataStax` は Amazon EC2 上に簡単にデプロイ出来るマルチノード Cassandra クラスタの AMI を提供する。

The DataStax AMI initializes all nodes in one availability zone using the SimpleSnitch.
If you want an EC2 cluster that spans multiple regions and availability zones, do not use the DataStax AMI. Instead, install Cassandra on your EC2 instances as described in Installing Cassandra Debian packages, and then configure the cluster as a multiple data center cluster.

`DataStax` の提供する AMI はすべてのノードが一つの AZ を使う SimpleSnitch で初期化される。もし、複数のリージョン、AZ をまたぐ EC2 クラスタを求める場合には `DataStax AMI` は利用しないこと。

Use the following guidelines when setting up your cluster:

クラスタを設定する際には以下のガイドラインを利用すること。

Only use known AMI's from a trusted source. Random AMI's pose a security risk and may perform levels slower than expected due to the way the install is configured for EC2. For example:

以下のような、信頼された既知の AMI だけを利用しまししょう。適当に選んだ AMI はセキュリティリスクをもたらしますし EC2 用の設定されているインストール方法で予想以上に遅い結果をもたらす原因となる。

 * Ubuntu Amazon EC2 AMI Locator
 * Debian AmazonEC2Image
 * CentOS-6 images on Amazon's EC2 Cloud

For production Cassandra clusters on EC2, use Large or Extra Large instances with local storage.

プロダクション環境の EC2 上の Cassandra cluster は Large 又は Extra Large インスタンスとローカルストレージを使う。

Amazon Web Service has reduced the number of default ephemeral disks attached to the image from four to two. Performance will be slower for new nodes unless you manually attach the additional two disks; see Amazon EC2 Instance Store.

AWS はデフォルトの `ephemeral disk` の数を 4 つから 2 つに削減した。さらに 2 つのディスクを手動でアタッチしない場合、パフォーマンスは新しいノードの為に遅くなるでしょう。
詳しいことは EC2 Instance Store を見てください。

RAID 0 the ephemeral disks, and put both the data directory and the commit log on that volume. This has proved to be better in practice than putting the commit log on the root volume (which is also a shared resource). For more data redundancy, consider deploying your Cassandra cluster across multiple availability zones or using EBS volumes to store your Cassandra backup files.

`data directory` と `commit log` 両方を`RAID0` の `ephemeral disk` のボリュームに置く。これは（これも共有リソースである）root ボリュームに `commit log` を置くよりも優れていることが証明されている。より多くのデータの冗長性を確保する為、複数の　AZ にまたがって Cassndra クラスタを展開したり、Cassandra のバックアップファイルを保存する為の EBS ボリュームの使用を検討すること。

Cassandra JBOD support allows you to use standard disks, but you may get better throughput with RAID0. RAID0 splits every block to be on another device so that writes are written in parallel fashion instead of written serially on disk.

Cassandra JBOD support を標準のディスクを使用することができますが、RAID0と利用するとより良いスループットを得ることができます。 RAID0は書き込みが並行して書かれたの代わりに、ディスク上に連続的に書き込まれるように、すべてのブロックが別のデバイス上にあるように分割します。

RAID0 は全てのブロックが別のデバイス上にあるように分割して書き込む。並行して書き込むようにしてディスク上に連続的に書き込むようにする。

EBS volumes are not recommended for Cassandra data volumes for the following reasons:
EBS volumes contend directly for network throughput with standard packets. This means that EBS throughput is likely to fail if you saturate a network link.
EBS volumes have unreliable performance. I/O performance can be exceptionally slow, causing the system to back load reads and writes until the entire cluster becomes unresponsive.
Adding capacity by increasing the number of EBS volumes per host does not scale. You can easily surpass the ability of the system to keep effective buffer caches and concurrently serve requests for all of the data it is responsible for managing.
For more information and graphs related to ephemeral versus EBS performance, see the blog article Systematic Look at EC2 I/O.