1. 開発環境を作成
	* CentOSのインストール
		isoを落としてきてVirtualBoxを使ってインストール
	* ネットワークにつながるようにする
		eth0のONBOOTをyesにしとく
	* Development tools と git vimをインストール
		yumコマンド
        yum groupinstall "Development tools"
        yum install git vim-enhanced nkf readline-devel wget curl man
	* sshで作業アカウントに接続
		外部から接続できるように、virtualboxからネットワークアダプターを追加（ホストオンリーアダプター）
		(IPadress 192.168.---はローカルのIP)

2. Perlの開発環境
	* plenvのインストール(Perlのインストール、管理するやつ)
		git clone https://github.com/tokuhirom/plenv.git ~/.plenv #httpsにしないとだめだった
		~/.bash_profileにplenvのパスを通して、plenv初期化して、シェルを再起動、
		perl-buildをインストール
		git clone https://github.com/tokuhirom/Perl-Build.git ~/.plenv/plugins/perl-build/
	* Perlのインストール
		バージョン5.20.2をインストールする plenv install 5.20.2
		使用するバージョンを指定する plenv global 5.20.2
		cpanmをインストールする plenv install-cpanm

3. 開発用データベースMySQL
	* dev.mysql.comからrpmを落としてきて入れる
		wget -q http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-client-5.6.11-1.el6.x86_64.rpm みたいな
		yum localinstall MySQL-*-5.6.25-1.el6.x86_64.rpm みたいな
		cat /root/.mysql_secret にrootに初期パスワードが設定されている
		/etc/init.d/mysql start でMySQLサーバースタート
		mysql -u root でログイン

4. リバースプロキシを設定
	* nginxのインストール
		CentOSのリポジトリにないから、	nginxがくれてるリポを追加
		touch /etc/yum.repos.d/nginx.repo
		中身↓
		[nginx]
		name=nginx repo
		baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
		gpgcheck=0
		enabled=1

		あとは普通にyum install nginx
		/etc/init.d/nginx start でOKなればよし

5. アプリケーションのデプロイ
	* cpanmでCatalyst環境を整える
		cpanm -n Task::Catalyst Task::Plack DBIx::Class::Schema::Loader Perl::Tidy Plack::Middleware::ReverseProxy
		cpanm --installdeps .  --->  cpanfileの中身をインストール?
		plackup -Ilib -e 'enable "Plack::Middleware::ReverseProxy";' bookmark_web.psgi
		---> .psgiを動かす?
			use Bookmark::Web;
			my $app = Bookmark::Web->apply_default_middlewares(Bookmark::Web->psgi_app);
			$app;
	* リバースプロキシを使う
		nginx.conf
			http{
			    server{
			        listen 80;
				    server_name 192.168.56.101
			        location / {
				    proxy_pass http://192.168.56.101:5000;
				    }
			    }
			}
		- listen nginxのポート番号を決める
		- proxy_pass どのplackup(サーバ側のやつ、処理する奴)に渡すか決める

6. nginxをソースから入れなおす
    * nginxをアンインストール
        yum remove nginx
    * ほしいバージョンのnginxをダウンロード
        --> wget http://---------
    * ./configue (～～～オプション)
        --> ./configue --help でオプションが見られる
            インストールする場所とか。デフォならusr/local？
    * make でMakefileをつくる
    * make install でインストール

7. nginxにLDAP認証を追加する
    * ldapのファイルを落としてくる
        - git かtar.gzで落としてきて展開
    * nginxのconfigureをもう一回
        - オプションに"--add-module=../nginx-auth-ldap"を追加
    * make -> make install
    * nginx.confを編集、ldapに関する記述を追加
        - --add-module=../nginx-auth-ldapl
            http {
                ....

                ldap_server test0{
                    url ldap://ldap.adways.net/dc=adways,dc=net?uid?sub?(objectClass=netAdwaysStaff);
                    binddn uid=apache,dc=adways,dc=net;
                    binddn_passwd apache;
                    require valid_user;
                }
                server {
                    listen 80;
                    server_name 192.168.56.101;
                    ....
                    location / {
                        auth_ldap "LDAP";
                        auth_ldap_servers test0;
                        proxy_pass http://192.168.56.101:5000;
                    }
                }
            }
    * ※nginx.pidってのがないぞって怒られたので、touchしたら動いた。
    * ※configureのオプションよくわからんかったからこぴぺ
        sudo ./configure \
            --prefix=/etc/nginx/ \
            --sbin-path=/usr/sbin/nginx \
            --conf-path=/etc/nginx/nginx.conf \
            --error-log-path=/var/log/nginx/error.log \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --http-log-path=/var/log/nginx/access.log \
            --http-client-body-temp-path=/var/cache/nginx/client_temp \
            --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
            --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
            --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
            --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
            --with-http_ssl_module \
            --with-http_realip_module \
            --with-ipv6 \
            --add-module=../nginx-auth-ldap \
            --with-http_ssl_module
