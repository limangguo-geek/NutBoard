# NutBoard 安装部署教程

## 环境要求

- PHP
- MySQL
- Nginx
- PHP 拓展：
  - Redis
  - fileinfo
  - igbinary

## PHP 配置

### 解除禁用函数

为了保证 NutBoard 的正常运行，需要解除以下 PHP 函数的禁用：

- `putenv`
- `proc_open`
- `pcntl_alarm`
- `pcntl_signal`

## 安装步骤

1.  **清空网站目录并克隆代码：**

    首先，清空你的网站根目录，然后使用 Git 克隆 NutBoard 代码仓库：

    ```bash
    git clone https://github.com/limangguo-geek/NutBoard.git ./
    ```

2.  **运行初始化脚本：**

    克隆完成后，运行初始化脚本以完成基本配置：

    ```bash
    sh init.sh
    ```

## Nginx 配置

根据你的需求选择以下配置方案：

### 方案一：使用 Webman（推荐）

如果你使用 Webman，请使用以下 Nginx 配置：

```nginx
location /downloads {
}

location / {
    try_files $uri $uri/ @backend;
}

location ~ (/config/|/manage/|/webhook|/payment|/order|/theme/) {
    try_files $uri $uri/ /index.php$is_args$query_string;
}

location @backend {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:6600;
}

location ~ .*\.(js|css)?$ {
    expires 1h;
    error_log off;
    access_log /dev/null; 
}
```

### 方案二：标准模式

如果你不使用 Webman，请使用以下 Nginx 配置：

```nginx
location /downloads {
}

location / {  
    try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$ {
    expires 1h;
    error_log off;
    access_log /dev/null; 
}
```

## 系统配置

### 配置定时任务

使用以下命令配置定时任务：

```bash
php /www/wwwroot/xxxx/artisan schedule:run
```

请注意替换 `/www/wwwroot/xxxx` 为你的实际网站根目录。

### 配置进程守护

为了保证 NutBoard 持续运行，你需要配置进程守护。根据你选择的运行模式，选择以下命令：

-   **Webman 模式：**

    ```bash
    php -c cli-php.ini webman.php start
    ```

-   **其他模式：**

    ```bash
    php artisan horizon
    ```

## 迁移教程

### 更新代码

```bash
git remote set-url origin https://github.com/limangguo-geek/NutBoard
git checkout master
./update.sh
```

### 重启队列服务

更新代码后，需要重启队列服务以应用新的更改：

```bash
php artisan config:clear
php artisan config:cache
php artisan horizon:terminate
```