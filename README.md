# This is a repository like a tutorial about functions in PostgreSQL

I wanted to leave here my some knowladges and experiences with functions/procedures PostgreSQL.
## Acknowledgements

Recently, I have worked with a project like [Trello](https://trello.com/). All tables and functions these i created are here 👇👇👇
#

## Tables

```bash
                List of relations
 Schema  |      Name      | Type  |  Owner
---------+----------------+-------+----------
 todoapp | project        | table | postgres
 todoapp | project_column | table | postgres
 todoapp | project_member | table | postgres
 todoapp | task_comment   | table | postgres
 todoapp | task_member    | table | postgres
 todoapp | tasks          | table | postgres
 todoapp | users          | table | postgres
```

#

users table

```bash
   Column   |           Type           | Collation | Nullable |              Default
------------+--------------------------+-----------+----------+-----------------------------------
 id         | bigint                   |           | not null | nextval('users_id_seq'::regclass)
 username   | character varying(50)    |           | not null |
 password   | character varying(100)   |           | not null |
 email      | character varying(50)    |           | not null |
 role       | character varying(50)    |           |          | 'USER'::character varying
 created_at | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_at | timestamp with time zone |           |          |
 language   | character varying        |           | not null | 'RU'::character varying
 is_deleted | smallint                 |           | not null | 0
```
#

project table

```bash
   Column    |           Type           | Collation | Nullable |               Default
-------------+--------------------------+-----------+----------+-------------------------------------
 id          | bigint                   |           | not null | nextval('project_id_seq'::regclass)
 title       | character varying        |           | not null |
 description | character varying        |           | not null |
 completed   | boolean                  |           | not null | false
 is_deleted  | boolean                  |           | not null | false
 created_by  | bigint                   |           |          |
 created_at  | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_by  | bigint                   |           |          |
 updated_at  | timestamp with time zone |           |          |
 code        | character varying        |           | not null |
```
#

tasks table

```bash
      Column       |           Type           | Collation | Nullable |               Default
-------------------+--------------------------+-----------+----------+--------------------------------------
 id                | bigint                   |           | not null | nextval('tasks_id_seq'::regclass)
 title             | character varying        |           | not null |
 description       | character varying        |           |          |
 project_column_id | bigint                   |           |          |
 priority          | character varying        |           | not null | 'LOW'::character varying
 level             | character varying        |           | not null | 'EASY'::character varying
 order             | smallint                 |           | not null | nextval('tasks_order_seq'::regclass)
 is_deleted        | boolean                  |           | not null | false
 created_by        | bigint                   |           |          |
 created_at        | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_by        | bigint                   |           |          |
 updated_at        | timestamp with time zone |           |          |
```
#

task_member table

```bash
 Column  |  Type  | Collation | Nullable |                 Default
---------+--------+-----------+----------+-----------------------------------------
 id      | bigint |           | not null | nextval('task_member_id_seq'::regclass)
 task_id | bigint |           |          |
 user_id | bigint |           |          |
```

#

task_comment table

```bash
    Column    |           Type           | Collation | Nullable |                 Default
--------------+--------------------------+-----------+----------+------------------------------------------
 id           | bigint                   |           | not null | nextval('task_comment_id_seq'::regclass)
 task_id      | bigint                   |           |          |
 message      | character varying        |           | not null |
 comment_type | character varying        |           | not null | 'MESSAGE'::character varying
 is_deleted   | boolean                  |           | not null | false
 created_by   | bigint                   |           |          |
 created_at   | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_by   | bigint                   |           |          |
 updated_at   | timestamp with time zone |           |          |
```

#

project_member table

```bash
   Column   |           Type           | Collation | Nullable |                  Default
------------+--------------------------+-----------+----------+--------------------------------------------
 id         | bigint                   |           | not null | nextval('project_member_id_seq'::regclass)
 user_id    | bigint                   |           |          |
 project_id | bigint                   |           |          |
 is_lead    | boolean                  |           | not null | false
 is_deleted | boolean                  |           | not null | false
 created_by | bigint                   |           |          |
 created_at | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_by | bigint                   |           |          |
 updated_at | timestamp with time zone |           |          |
```

#
project_column table

```bash
      Column       |           Type           | Collation | Nullable |               Default
-------------------+--------------------------+-----------+----------+--------------------------------------
 id                | bigint                   |           | not null | nextval('tasks_id_seq'::regclass)
 title             | character varying        |           | not null |
 description       | character varying        |           |          |
 project_column_id | bigint                   |           |          |
 priority          | character varying        |           | not null | 'LOW'::character varying
 level             | character varying        |           | not null | 'EASY'::character varying
 order             | smallint                 |           | not null | nextval('tasks_order_seq'::regclass)
 is_deleted        | boolean                  |           | not null | false
 created_by        | bigint                   |           |          |
 created_at        | timestamp with time zone |           | not null | CURRENT_TIMESTAMP
 updated_by        | bigint                   |           |          |
 updated_at        | timestamp with time zone |           |          |
```

## Functions

# 
*Email checker*
```bash
create function check_email(email character varying DEFAULT NULL::character varying) returns boolean
    language plpgsql
as
$$
declare
    pattern varchar := '^([a-zA-Z0-9_\-\.]+)@([a-zA-Z0-9_\-]+)(\.[a-zA-Z]{2,5}){1,2}$';
BEGIN
    return email ~* pattern;
END
$$;
```
# 
*Password encoder*
```bash
create function encode_password(rawpassword character varying) returns character varying
    language plpgsql
as
$$
begin
    if rawPassword is null then
        raise exception 'Invalid Password null';
    end if;
    return utils.crypt(rawPassword, utils.gen_salt('bf', 4));
end
$$;
```

# 
*Checker password matching*
```bash
create function match_password(rawpassword character varying, encodedpassword character varying) returns boolean
    language plpgsql
as
$$
declare

begin
    if rawPassword is null then
        raise exception 'Invalid Password null';
    end if;

    if encodedPassword is null then
        raise exception 'Invalid encoded Password null';
    end if;
    return encodedPassword = utils.crypt(rawPassword, encodedPassword);
end
$$;
```

# 
*is Active*
```bash
create procedure isactive(IN userid bigint DEFAULT NULL::bigint)
    language plpgsql
as
$$
declare
    t_user record;
BEGIN
    if userid is null then
        raise exception 'User id is null';
    end if;

    select * into t_user from users t where t.is_deleted = 0 and t.id = userid;
    if not FOUND then
        raise exception 'User not found by id : ''%''',userid;
    end if;
END
$$;
```
# 
*has Role*
```bash
create function hasrole(userid bigint DEFAULT NULL::bigint, role character varying DEFAULT NULL::character varying) returns boolean
    language plpgsql
as
$$
declare
    t_user record;
BEGIN
    if userid is null or role is null then
        return false;
    end if;
    select * into t_user from users t where t.is_deleted = 0 and t.id = userid;
    return FOUND and t_user.role = role;
END
$$;
```
# 
*register*
```bash
create function auth_register(dataparam text) returns integer
    language plpgsql
as
$$
declare
    newId    int4;
    dataJson json;
    t_user   record;
    v_dto    todoapp.auth_register_dto;
begin
    if dataparam isnull or dataparam = '{}'::text then
        raise exception 'Data param can not be null';
    end if;

    dataJson := dataparam::json;
    v_dto := mappers.json_to_auth_register_dto(dataJson);

    if v_dto.username is null or trim(v_dto.username) = '' then
        raise exception 'Username is invalid';
    end if;

    if v_dto.email is null or trim(v_dto.email) = '' then
        raise exception 'Email is invalid';
    end if;


    if utils.check_email(v_dto.email) is false then
        raise exception 'Email is invalid';
    end if;

    select * into t_user from todoapp.users t where t.username ilike v_dto.username and is_deleted = 0;
    if FOUND then
        raise exception 'Username ''%'' already taken',t_user.username;
    end if;

    select * into t_user from todoapp.users t where t.email ilike v_dto.email and is_deleted = 0;
    if FOUND then
        raise exception 'Email ''%'' already taken',t_user.email;
    end if;

    if v_dto.password is null or trim(v_dto.password) = '' then
        raise exception 'Password is invalid';
    end if;

    insert into todoapp.users (username, password, email, role)
    values (v_dto.username,
            utils.encode_password(v_dto.password),
            v_dto.email,
            v_dto.role)
    returning id into newId;
    return newId;
end
$$;
```
# 
*login*
```bash
create function auth_login(uname character varying DEFAULT NULL::character varying, pswd character varying DEFAULT NULL::character varying) returns text
    language plpgsql
as
$$
declare
    t_user record;
begin

    select * into t_user from todoapp.users t where t.username ilike uname and is_deleted  = 0;
    if not FOUND then
        raise exception 'User not found by username ''%''',uname;
    end if;

    if utils.match_password(pswd, t_user.password) is false then
        raise exception 'Bad credentials';
    end if;
    return json_build_object('id', t_user.id,
                               'username', t_user.username,
                               'email', t_user.email,
                               'language', t_user.language,
                               'position', t_user.role,
                               'created_at', t_user.created_at,
                               'updated_at', t_user.updated_at)::text;

end
$$;
```

# 
*uodate user*
```bash
create function auth_user_update(dataparam character varying DEFAULT NULL::character varying, userid bigint DEFAULT NULL::bigint) returns boolean
    language plpgsql
as
$$
declare
    dataJson   json;
    t_user     record;
    v_username varchar;
    v_email    varchar;
    v_role     varchar;
    v_language varchar;
    v_id       bigint;
begin

    call todoapp.isactive(userid);
    
    if dataparam is null or dataparam = '{}'::text then
        raise exception 'Dataparam can not be null';
    end if;

    dataJson := dataparam::json;

    v_id := dataJson ->> 'id';
    v_username := dataJson ->> 'username';
    v_email := dataJson ->> 'email';
    v_language := dataJson ->> 'language';
    v_role := dataJson ->> 'role';

    if v_id != userid and todoapp.hasRole(userid, 'ADMIN') is false then
        raise exception 'Permission denied';
    end if;
    -- TODO check username, password, email, role

    if utils.check_email(v_email) is false then
        raise exception 'Email invalid ''%''', v_email;
    end if;

    select * into t_user from todoapp.users t where t.is_deleted = 0 and t.id = v_id;
    if not FOUND then
        raise exception 'User not found by id ''%''',v_id;
    end if;

    if v_username is null then
        v_username := t_user.username;
    end if;
    if v_email is null then
        v_email := t_user.email;
    end if;
    if v_role is null then
        v_role := t_user.role;
    end if;
    if v_language is null then
        v_language := t_user.language;
    end if;

    update todoapp.users
    set username = v_username,
        role     = v_role,
        language = v_language,
        email    = v_email
    where id = v_id;

    return true;
end
$$;
```

# 
*user info*
```bash
create function userinfo(userid bigint DEFAULT NULL::bigint) returns jsonb
    language plpgsql
as
$$
declare
    r_user record;
begin
    select * into r_user from todoapp.users u where u.is_deleted = 0 and u.id = userid;
    if FOUND then
        return row_to_json(X)::jsonb
            FROM (SELECT r_user.id, r_user.username, r_user.email, r_user.language, r_user.role) AS X;
    else
        return null;
    end if;
end
$$;
```

# 
*project column details*
```bash
create function project_column_details(pc todoapp.users DEFAULT NULL::todoapp.users) returns jsonb
    language plpgsql
as
$$
declare

begin
    if pc is null then
        return null;
    else
        return row_to_json(X)::jsonb FROM (SELECT pc.id, pc.name, pc.order) AS X;
    end if;
end
$$;
```

# 
*column tasks*
```bash
create function project_column_tasks(pcid bigint DEFAULT NULL::bigint) returns json
    language plpgsql
as
$$
declare

begin
    return (select json_agg(json_build_object(
            'id', t.id,
            'title', t.title,
            'description', t.description,
            'priority', t.priority,
            'level', t.level,
            'order', t."order",
            'createdBy', todoapp.userInfo(t.created_by),
            'created_at', t.created_at,
            'members', todoapp.task_members(t.id),
            'comments', todoapp.task_comments(t.id)
        ))
            from todoapp.tasks t
            where not t.is_deleted
              and t.project_column_id = pcid);
end
$$;
```
# 
*project create*
```bash
create function project_create(dataparam text DEFAULT NULL::text, userid bigint DEFAULT NULL::bigint) returns bigint
    language plpgsql
as
$$

declare
    dataJson json;
    newId    bigint;

begin
    call todoapp.isactive(userid);

    if dataparam is null or dataparam = '{}'::text then
        raise exception 'Datapram invalid';
    end if;

    dataJson := cast(dataparam as json);

    if exists(select * from todoapp.project t where not is_deleted and t.code = upper(dataJson ->> 'code')) then
        raise exception 'Project with code ''%'' already exists', dataJson ->> 'code';
    end if;

    insert into todoapp.project(title, code, description, created_by)
    values (dataJson ->> 'title',
            upper(dataJson ->> 'code'),
            dataJson ->> 'description',
            userid)
    returning id into newId;

    insert into todoapp.project_column (name, "order", project_id, created_by)
    values ('TODO', nextval('todoapp.project_column_order_seq'), newId, userid),
           ('DOING', nextval('todoapp.project_column_order_seq'), newId, userid),
           ('DONE', nextval('todoapp.project_column_order_seq'), newId, userid);
    
    insert into todoapp.project_member (user_id, project_id, is_lead, created_by)
    values (userid, newId, true, userid);
    
    return newId;
end;
$$;
```
# 
*project details*
```bash
create function project_details(projectid bigint DEFAULT NULL::bigint, userid bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
declare
    result      json;
    membersInfo json;
    columnsInfo json;
    r_project   record;
begin
    CALL todoapp.isactive(userid);
    select * into r_project from todoapp.project t where not t.is_deleted and t.id = projectid;

    if not FOUND then
        raise exception 'Project not found by id ''%'' ', projectid;
    end if;


    membersInfo := json_agg(json_build_object(
            'member_id', t.id,
            'userInfo', todoapp.userInfo(t.user_id)
        ))
                   from todoapp.project_member t
                   where not t.is_deleted
                     and t.project_id = projectid;


    columnsInfo := json_agg(json_build_object(
            'column_id', pc.id,
            'name', pc.name,
            'position', pc.position,
            'tasks', todoapp.project_column_tasks(pc.id)
        ))
                   from todoapp.project_column pc
                   where not pc.is_deleted
                     and pc.project_id = projectid;

    result := json_build_object(
            'id', r_project.id,
            'title', r_project.title,
            'code', r_project.code,
            'description', r_project.description,
            'completed', r_project.completed,
            'created_at', r_project.created_at,
            'created_by', r_project.created_by,
            'columns', columnsInfo,
            'members', membersInfo
        );
    return result::text;
end
$$;
```
# 
*add member to project*
```bash
create function project_member_add(dataparam character varying DEFAULT NULL::character varying) returns boolean
    language plpgsql
as
$$
<<out>>
    declare
    dataJson json;
    dto      todoapp.projectmembercreatedto;
    member  todoapp.userid_and_islead;
begin
    if dataparam is null or dataparam = '{}'::text or dataparam = ''::text then
        raise exception 'Data param invalid';
    end if;

    dataJson := dataparam::json;
    dto.project_id = dataJson ->> 'project_id';

    if dto.project_id is null then
        raise exception 'Project id can not be null';
    end if;

    if not exists(select * from todoapp.project t where not t.is_deleted and t.id = dto.project_id) then
        raise exception 'Project not found with id "%"',dto.project_id;
    end if;
    for member in select json_array_elements(dataJson -> 'members')
        loop
            if exists(select * from todoapp.users t where t.is_deleted = 0 and t.id = member.user_id) then
                insert into todoapp.project_member (user_id, project_id,is_lead) values (member.user_id,dto.project_id,member.islead );
            end if;
        end loop;
    return true;
end
$$;
```
# 
*task comments*
```bash
create function task_comments(taskid bigint DEFAULT NULL::bigint) returns json
    language plpgsql
as
$$
begin
    return coalesce((select json_agg(json_build_object(
            'message', t.message,
            'type', t.comment_type,
            'userInfo', todoapp.userInfo(t.created_by)))
                     from todoapp.task_comment t
                     where t.task_id = taskid), '[]');


end
$$;
```
# 
*task create*
```bash
create function task_create(dataparam character varying, userid bigint) returns bigint
    language plpgsql
as
$$
declare
    dataJson json;
    newId    bigint;
    dto      todoapp.taskcreatedto;
begin

    if dataparam is null or dataparam = '{}'::text or dataparam = ''::text then
        raise exception 'Data param is invalid';
    end if;

    call todoapp.isactive(userid);

    dataJson := dataparam::json;
    dto.title := dataJson ->> 'title';
    dto.description := dataJson ->> 'description';
    dto.project_column_id := dataJson ->> 'project_column_id';
    dto.members := dataJson ->> 'members';

    if dto.title is null then
        raise exception 'Title can not be null';
    end if;

    if dto.project_column_id is null then
        raise exception 'Title can not be null';
    end if;

    if not exists(select * from todoapp.project_column t where not t.is_deleted and t.id = dto.project_column_id) then
        raise exception 'Project column not found';
    end if;

    insert into todoapp.tasks(title, description, project_column_id, created_by)
    values (dto.title, dto.description, dto.project_column_id, userid)
    returning id into newId;


    return newId;
end
$$;
```
# 
*add member to task*
```bash
create function task_member_add(dataparam character varying DEFAULT NULL::character varying) returns boolean
    language plpgsql
as
$$
<<out>>
    declare
    dataJson json;
    dto      todoapp.taskmembercreatedto;
    user_id  bigint;
begin


    if dataparam is null or dataparam = '{}'::text or dataparam = ''::text then
        raise exception 'Data param invalid';
    end if;

    dataJson := dataparam::json;
    dto.task_id = dataJson ->> 'task_id';
    raise info '%', dataparam;


    if dto.task_id is null then
        raise exception 'Task id can not be null';
    end if;

    if not exists(select * from todoapp.tasks t where not t.is_deleted and t.id = dto.task_id) then
        raise exception 'Task not found with id "%"',dto.task_id;
    end if;

/*    if dto.members is null or array_length(dto.members, 1) = 0 then
        raise exception 'Members can not be null';
    end if;*/

    for user_id in select json_array_elements(dataJson -> 'members')
    loop
        if exists(select * from todoapp.users t where t.is_deleted = 0 and t.id = user_id) then
            insert into todoapp.task_member (task_id, user_id) values (dto.task_id, user_id);
        end if;
    end loop;

    return true;
end
$$;
```
