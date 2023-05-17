sudo firewall-cmd --add-port=80/tcp --permanent

sudo systemctl restart firewalld

ssh-keygen -t rsa -b 2048

cat ~/.ssh/id_rsa.pub

Add SSH key to GitLab

Copy 'html' SSH

cd /var/www

git clone 'GitLab SSH'

cd /var/www/html

code -r .

other useful tools

su root
usermod -aG wheel stud3

cd ~/.ssh/
ssh-keygen -t rsa -b 2048
cat ~/.ssh/ (location)
cat ~/.ssh/ (location).pub
ssh-copy-id -i ~/.ssh/(file location).pub stud3@(ip address)
ssh stud3@(ip address)

