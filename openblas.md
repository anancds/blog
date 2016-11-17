# openblas库安装

直接用openblas库编译

git clone [https://github.com/xianyi/OpenBLAS.git](https://github.com/xianyi/OpenBLAS.git)
make clen
make -j4
make intall PREFIX=/root/openblas-install
ln -sf /root/openblas-install/lib/libopenblas.so  /usr/lib64/libblas.so
ln -sf /root/openblas-install/lib/libopenblas.so  /usr/lib64/libblas.so.3
ln -sf /root/openblas-install/lib/libopenblas.so  /usr/lib64/liblapack.so.3

spark编译命令：

dev/make-distribution.sh --name blas --tgz -Pyarn -Phadoop-2.4 -Dhadoop.version=2.5.2 -Pnetlib-lgpl

参考资料：

[https://github.com/amplab/ml-matrix](https://github.com/amplab/ml-matrix)
[https://github.com/shivaram/matrix-bench](https://github.com/shivaram/matrix-bench)
[http://stackoverflow.com/questions/37848216/how-to-configure-high-performance-blas-lapack-for-breeze-on-amazon-emr-ec2](http://stackoverflow.com/questions/37848216/how-to-configure-high-performance-blas-lapack-for-breeze-on-amazon-emr-ec2)
[https://www.cloudera.com/documentation/enterprise/5-7-x/topics/spark_mllib.html#concept_arw_q2j_h5](https://www.cloudera.com/documentation/enterprise/5-7-x/topics/spark_mllib.html#concept_arw_q2j_h5)
[http://www.spark.tc/blas-libraries-in-mllib/](http://www.spark.tc/blas-libraries-in-mllib/)

验证方式：

bin/run-example mllib.MovieLensALS --rank 5 --numIterations 5 --lambda 1.0 --kryo /home/sample_movielens_data.txt
