yum update -y
yum install -y docker amazon-efs-utils
systemctl enable --now docker
mkdir -p /mnt/efs
echo "fs-XXXX.efs.us-east-2.amazonaws.com:/ /mnt/efs efs _netdev,tls 0 0" >> /etc/fstab
mount -a
docker run -d --name wordpress -p 80:80 \
  -e WORDPRESS_DB_HOST=<endpoint-rds> \
  -e WORDPRESS_DB_NAME=name \
  -e WORDPRESS_DB_USER=admin \
  -e WORDPRESS_DB_PASSWORD=<senha> \
  -v /mnt/efs:/var/www/html \
  wordpress
