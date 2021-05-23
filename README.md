# dash_project
git clone https://github.com/Chris3691/Dash-User-Management

Already installed the required python packages with the following:
sudo pip3 install flask_login sqlalchemy flask_sqlalchemy PyMySQL


Edit the following file config.txt:
[database]
con = mysql+pymysql://user:password@localhost/database_name

Create a database in mysql with the following command:

CREATE DATABASE tgdata_db;
USE tgdata_db;
#Note password size has been changed to 88

CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) NOT NULL,
  `email` varchar(50) NOT NULL,
  `password` varchar(88) NOT NULL,
  `admin` tinyint(4) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `username_UNIQUE` (`username`),
  UNIQUE KEY `email_UNIQUE` (`email`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;


Edit the file users_mgt.py:
password = db.Column(db.String(80)) -> password = db.Column(db.String(88))

Either you use Mysql command to create the first admin user account or Edit the file index.py to create admin access to create first admin user:
Original Code:
```python
    if pathname == '/admin':
        if current_user.is_authenticated:
            if current_user.admin == 1:
                return user_admin.layout
            else:
                return error.layout
        else:
            return login.layout
```
Modified code (temporary) to create admin user:
```python
    if pathname == '/admin':
        if True #if current_user.is_authenticated:
            if True: #if current_user.admin == 1:
                return user_admin.layout
            else:
                return error.layout
        else:
            return login.layout
```
