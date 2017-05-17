# Cài đặt openresty kết hợp với galera sử dụng mariadb trong mô hình nginx


# Mục lục

- [Vấn đề bài toán và mô hình](#issue)
- [Cài đặt mariadb và galera](#install)
- [Cấu hình galera cho các node](#config)
- [Kiểm tra kết quả](#test)
- [Các nội dung khác](#content-others)


# Nội dung

- #### <a name="issue">Vấn đề bài toán và mô hình</a>

	+ Trong các mô hình website, sẽ có những trường hợp một máy chủ chứa database bị lỗi, không thể đáp ứng được cho hoạt động của ứng dụng sử dụng nó chẳng hạn như website. Để đảm bảo cho tính luôn sẵn sàng hoạt động ổn định nhất, ta cần phải có một database khác thay thế cho database cũ đang ngừng hoạt động để duy trì cho ứng dụng được tiếp tục hoạt động.

	+ Mô hình có thể được minh họa như sau:

		![openresty-nginx-galera](../images/openresty-nginx-galera/openresty-cluster-load-balancer.png)

	+ IP Planning như sau:
		
		![openresty-nginx-galera](../images/openresty-nginx-galera/ip-planning.png)

- #### <a name="install">Cài đặt mariadb và galera</a>

	##### Lưu ý:
			- Các câu lệnh được thực hiện trên OS: Centos 7
			- Chạy với quyền của superuser (root)

	+ Cài đặt Galera

		- Trên cả 2 node backends01, backends02, ta tiến hành chạy câu lệnh sau để thêm repository:

				# echo '[mariadb]
				 name = Mariadb
				 baseurl = http://yum.mariadb.org/10.1/centos7-amd64
				 gpgkey=https://yum.mariadb.org/rpm-gpg-key-mariadb
				 gpgcheck=1' >> /etc/yum.repos.d/mariadb.repo
		
		- Thực hiện cấu hình proxy để tăng tốc độ download cho cài đặt:
				# echo "proxy=http://123.30.178.220:3142" >> /etc/yum.conf

		- Tiến hành cài đặt mariadb, rsync:

				# yum install mariadb-server rsync

			trong câu lệnh trên, ta đã thực hiện cài đặt mariadb-server phiên bản 10.1.x nên không cần phải thực hiện cài đặt thêm galera. Bởi nó được cài đặt kèm theo.

		- Cấu hình trỏ host trong file */etc/hosts*:

				# echo "10.10.10.10      lb03" >> /etc/hosts
				  echo "10.10.10.20      backends01" >> /etc/hosts
				  echo "10.10.10.30      backends02" >> /etc/hosts

- #### <a name="config">Cấu hình galera cho các node</a>

	- Sau khi tiến hành cài đặt thành công mariadb-server và rsync, ta thực hiện chèn thêm nội dung vào file */etc/my.cnf.d/server.cnf* trên backends01:

			# vi /etc/my.cnf.d/server.cnf

		với nội dung sau trong mục *galera*:

			    # Mandatory settings
			    wsrep_on=ON
			    wsrep_provider=/usr/lib/galera/libgalera_smm.so
			    wsrep_cluster_name="linoxide-cluster"
			    wsrep_cluster_address="gcomm://backends01,backends02"

			    binlog_format=row
			    default_storage_engine=InnoDB
			    innodb_autoinc_lock_mode=2
			    #
			    # Allow server to accept connections on all interfaces.
			    #
			    bind-address=0.0.0.0
			    #
			    wsrep_sst_method=rsync

			    wsrep_node_address="10.10.10.20"
			    wsrep_node_name="mariadb01"
				wsrep_sst_method=rsync

	- Sau khi tiến hành cài đặt thành công mariadb-server và rsync, ta thực hiện chèn thêm nội dung vào file */etc/my.cnf.d/server.cnf* trên backends02:

			# vi /etc/my.cnf.d/server.cnf

		với nội dung sau trong mục *galera*:

			    # Mandatory settings
			    wsrep_on=ON
			    wsrep_provider=/usr/lib/galera/libgalera_smm.so
			    wsrep_cluster_name="linoxide-cluster"
			    wsrep_cluster_address="gcomm://backends01,backends02"

			    binlog_format=row
			    default_storage_engine=InnoDB
			    innodb_autoinc_lock_mode=2
			    #
			    # Allow server to accept connections on all interfaces.
			    #
			    bind-address=0.0.0.0
			    #
			    wsrep_sst_method=rsync

			    wsrep_node_address="10.10.10.30"
			    wsrep_node_name="mariadb02"
				wsrep_sst_method=rsync

	- Trên cả backends01 và backends02 ta thực hiện chạy câu lệnh sau:

			# systemctl stop firewalld
			# systemctl disable firewalld
			# systemctl stop mysql

			# firewall-cmd --permanent --add-port=3306/tcp
			# firewall-cmd --permanent --add-port=4567/tcp
			# firewall-cmd --permanent --add-port=873/tcp
			# firewall-cmd --permanent --add-port=4444/tcp
			# firewall-cmd --permanent --add-port=9200/tcp
			# firewall-cmd --reload
			# setenforce 0

	- Trên backends01 (hoặc backends02 - chỉ thực hiện lệnh này trên một backends duy nhất) ta thực hiện chạy câu lệnh sau để tạo ra một cluster mới:

			# galera_new_cluster

	- Trên backends còn lại, ta thực hiện lệnh sau để khởi động dịch vụ mariadb:

			# systemctl start mysql

	- Chuyển sang backends chạy câu lệnh tạo cluster mới. Ta thực hiện chạy câu lệnh sau để kiểm tra kết quả:

			# mysql -u root -e "SHOW STATUS LIKE 'wsrep_cluster_size'"

		Kết quả sẽ được hiển thị tương tự như sau:

			+---------------------+-----------+
			|  Variable_name      |   Value   |
			+---------------------+-----------+
			|  wsrep_cluster_size |     2     |
			+---------------------+-----------+

	- Trên backends01, ta thực hiện cài đặt maxsacle như sau:

			# yum install maxscale

	- Tiếp theo, sau khi maxscale đã được cài đặt, ta cần thực hiện cấp quyền truy cập database. Đầu tiên, ta cài đặt password cho user root để truy nhập database bằng việc chạy câu lệnh:

			# mysql_secure_installation

	- Cấp quyền truy cập database:

			# mysql -u root -p
			Enter password: 

	- Tạo người sử dụng:

			CREATE USER 'maxscale'@'%' IDENTIFIED BY 'password';
			GRANT SELECT ON mysql.db TO 'maxscale'@'%';
			GRANT SELECT ON mysql.user TO 'maxscale'@'%';
			GRANT SHOW DATABASES ON *.* TO 'maxscale'@'%';
			
- #### <a name=""></a>
- #### <a name=""></a>




- # <a name="content-others">Các nội dung khác</a>

	Sẽ cập nhật sau.

	+ [](#)