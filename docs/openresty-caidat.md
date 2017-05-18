# Báo cáo: Sơ lược về Openresty

# Mục lục

- [Giới thiệu về Openresty](#about)
- [Cài đặt](#install)
- [Kiểm thử cài đặt](#test)
- [Giới thiệu OpenResty Packages Manager](#opm)
- [Các nội dung khác](#content-others)


# Nội dung

- #### <a name="about">Giới thiệu về Openresty</a>

	+ [Openresty](https://openresty.org/en/) là một nền tảng web tích hợp các tiêu chuẩn của [Nginx](http://nginx.org/) core, [LuaJIT](http://luajit.org/luajit.html) cùng với rất nhiều thư viện viết bởi *Lua* một các cần thận cùng các modules của Nginx chất lượng cao do cộng đồng phát triển. Được thiết kế để giúp các nhà phát triển dễ dàng có thể xây dựng mở rộng các ứng dụng web, các dịch vụ web và các cổng web động.

	+ Bằng cachs tận dụng các modules đã được thiết kế cho Nginx, Openresty dễ dàng trở thành một server nginx chạy các ứng dụng web mạnh mẽ mà trong đó, ta có thể sử dụng lua script cho việc cấu hình nginx để xây dựng nên một ứng dụng web mạnh mẽ có khả năng xử lý các kết nối 10K ~ 1000K trong cùng một server.

	+ *Openresty* hướng tới mục đích để chạy các ứng dụng web phía server hoàn toàn trong máy chủ nginx. Tận dụng các mô hình của nginx để thực hiện non-blocking I/O không chỉ với khách hàng mà còn có cả các backends từ xa như MySQL, Memcached và Redis ... bởi bản thân *Openresty* đã là một nginx server.

- #### <a name="install">Cài đặt</a>
	
	###### Lưu ý:
		- Các cài đặt được thực hiện trên OS: Centos 7
		- Câu lệnh được chạy với quyền superuser

	- ##### Thêm *openresty* repository tới OS để có thể cài đặt các gói cài đặt khác và nhận update trong tương lai.

		+ Tạo một file */etc/yum.repos.d/OpenResty.repo*
				# touch /etc/yum.repos.d/OpenResty.repo

		+ Thêm nội dung sau vào file sau đó lưu lại:

				[openresty]
				name=Official OpenResty Repository
				baseurl=https://copr-be.cloud.fedoraproject.org/results/openresty/openresty/epel-$releasever-$basearch/
				skip_if_unavailable=True
				gpgcheck=1
				gpgkey=https://copr-be.cloud.fedoraproject.org/results/openresty/openresty/pubkey.gpg
				enabled=1
				enabled_metadata=1

		- Thực hiện cấu hình proxy để tăng tốc độ download cho cài đặt:
				# echo "proxy=http://123.30.178.220:3142" >> /etc/yum.conf
				
		+ Cài đặt *openresy* bằng cách chạy câu lệnh

				# yum install openresty

		+ Cài đặt thêm các gói cài đặt khác để có thể dễ dàng quản lý openresty:

				# yum install openresty-*.noarch 

		+ Bạn cũng có thể xem thêm các gói cài đặt khác có thể đi kèm với openresty bằng việc chạy câu lệnh sau:

				# sudo yum --disablerepo="*" --enablerepo="openresty" list available


- #### <a name="test">Kiểm thử cài đặt</a>

	+ Để tiến hành kiểm thử xem đã cài đặt thành công hay chưa, ta thực hiện chạy câu lệnh sau:

			# resty -e "print('Helo World')"
			Helo World

		Nếu kết quả được in ra thì ta đã thực hiện cài đặt thành công openresty.

	+ Thực hiện câu lệnh sau để kiểm tra thực sự xem  openresty đã hoạt động hay chưa: 

			# curl http://ip_os:8080

	+ Bạn có thể cấu hình lại cho openresty (nginx) tại thư mục: */etc/local/openresty/nginx/conf*

- # <a name="opm">Giới thiệu OpenResty Packages Manager</a>

	+ opm (OpenResty Packages Manager) là một công cụ cho phép quản lý các packages mà ta đã cài đặt thêm cho openresty ngoài những modules chính của nó để phát triển webserver. Đây là công cụ đã được cài đặt đi kèm theo hướng dẫn ở phía trên trong phần cài đặt openresty.

	+ Để hiển thị các gợi ý sử dụng opm, ta chạy câu lệnh sau:

			# opm --help

	+ Để tìm kiếm một packages nào đó hỗ trợ cho openresty, ta dùng câu lệnh

			# opm search	package's_name

		ví dụ:
			# opm search	lua-resty-cookie
			p0pr0ck5/lua-resty-cookie		Lua library for HTTP cookie manipulations for OpenResty/ngx_lua

	+ Để thực hiện cài đặt packages, ta chạy câu lệnh:

			# opm get package's_name


- # <a name="content-others">Các nội dung khác</a>

	Sẽ cập nhật sau.

	+ [](#)