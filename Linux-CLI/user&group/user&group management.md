# 用户管理相关

参考：https://www.cnblogs.com/xd502djj/archive/2011/11/23/2260094.html

系统：ubuntu-20.04 focal

### 用户相关

#### 创建用户

1. 创建一个有用户自己文件夹的用户

   ```sh
   sudo adduser <新用户名>
   ```

   

2. 仅添加用户信息，不自动生成用户的所属根文件夹

   ```sh
   sudo useradd <新用户名>
   ```

   

3. 修改用户密码

   ```sh
   sudo passwd <用户名>
   ```

   

4. a

5. a

6. a

7. a

8. a

9. 

#### 删除用户

1. 删除`/etc/passwd`、`/etc/shadow`、`/etc/group/`、`/etc/gshadow`四个文件记录的账户和组信息

   ```sh
   sudo userdel <用户名>
   ```

   

2. 删除用户及组信息且删除用户已有的文件夹下所有内容

   ```sh
   # 直接删除相关所有内容
   sudo deluser <用户名>
   # 也可以用userdel加-r参数进行全部删除
   sudo userdel -r <用户名>
   ```

   

3. user





### 工作组

#### 创建工作组

```sh
sudo groupadd <组名>
```

#### 删除工作组

可能由于组中仍有用户删除失败

```sh
sudo groupdel <组名>
```



### 用户与工作组操作

#### 将用户添加进工作组

```sh
# 将用户从其他组中删除，并只加入这一个工作组
sudo usermod -G <组名> <用户名>
# 将用户的工作组再增加一个，不覆盖以前加入的组
sudo usermod -aG <组名> <用户名>
```

#### 将用户从工作组里面移除

```sh
sudo gpasswd -d <用户名> <组名>
```

#### 查看用户属于哪个组

```sh
groups <用户名>
```

#### 查看组里有哪些用户

```sh
#查看以其为主属组的用户
grep `grep oinstall /etc/group | cut -d ":" -f 3 ` /etc/passwd | cut -d ":" -f 1
#查看以其为附加组的用户
grep oinstall /etc/group | cut -d ":" -f 4 
```