# keystone client
Authenticating Using Sessions
```
from keystoneauth1.identity import v3
from keystoneauth1 import session
from keystoneclient.v3 import client

auth = v3.Password(auth_url="http://192.168.40.146:5000/v3",username="admin",password="ok123", project_name="admin",user_domain_id="default", project_domain_id="default")
sess = session.Session(auth=auth)
keystone = client.Client(session=sess)
```
## token (lưu trong memcached)
```
# lấy token
token = sess.get_token(auth)

# token data
keystone.tokens.get_token_data(token)
```
## tạo nova_client bằng token
```
cli_nova.Client(2, auth_url='http://192.168.40.146:5000/v3', auth_token=token, project_name='admin', project_domain_id='default')
```

## Thao tác với user thông qua user admin
```
# lấy danh sách user list(domain, group, default_project, project) 
keystone.users.list()

# lấy user theo name, id, ...
keystone.users.find(name='dungnv')

# tạo user: create(*name, *password, *domain, *project, email, description, default_project, enable)
keystone.users.create(name='mticket', password='ok123', domain='default', projects='dung123')

# cập nhật: update(*user, name, password, domain, project, email, description, default_project, enable)
user = keystone.users.find(name='dungnv')
keystone.users.update(user=user, name='dungabc')

# xóa: delete(*user)
user = keystone.users.find(name='dungabc')
keystone.users.delete(user)
```

# tạo project
```
project = keystone.projects.find(name='test')
keystone.projects.create()
```
# tạo roles
```
keystone.roles.create()
```

