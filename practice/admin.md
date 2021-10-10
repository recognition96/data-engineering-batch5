## 스키마 생성

```
CREATE SCHEMA raw_data;
CREATE SCHEMA analytics;
CREATE SCHEMA adhoc;
```

## 그룹 생성과 권한 지정

```
CREATE GROUP analytics_users;
GRANT USAGE ON SCHEMA analytics TO GROUP analytics_users;
GRANT USAGE ON SCHEMA raw_data TO GROUP analytics_users;
GRANT SELECT ON ALL TABLES IN SCHEMA analytics TO GROUP analytics_users;
GRANT SELECT ON ALL TABLES IN SCHEMA raw_data TO GROUP analytics_users;
GRANT ALL ON SCHEMA adhoc to GROUP analytics_users;
GRANT ALL ON ALL TABLES IN SCHEMA adhoc TO GROUP analytics_users;
```

## 일반 사용자 생성과 권한 지정

```
CREATE USER keeyong PASSWORD '...';
ALTER GROUP analytics_users ADD USER keeyong;
CREATE SCHEMA keeyong;
ALTER SCHEMA keeyong OWNER TO keeyong;
```

## 테이블 생성과 복사

```
CREATE TABLE raw_data.user_session_channel ( 
    userId int,
    sessionId varchar(32) primary key,
    channel varchar(32)
);

CREATE TABLE raw_data.session_timestamp ( 
    sessionId varchar(32) primary key,
    ts timestamp
);

CREATE TABLE raw_data.session_transaction (
    sessionId varchar(32) primary key,
    refunded boolean,
    amount int
);

CREATE TABLE raw_data.channel (
    channelName varchar(32) primary key
);
```

```
COPY raw_data.channel
FROM 's3://..../channel.csv'
credentials 'aws_iam_role=arn:aws:iam::...'
delimiter ',' FORMAT AS CSV COMPUPDATE ON dateformat 'auto' timeformat 'auto' NULL AS 'NULL' IGNOREHEADER 1; 

GRANT SELECT ON TABLE raw_data.user_session_channel TO GROUP analytics_users;
GRANT SELECT ON TABLE raw_data.session_timestamp TO GROUP analytics_users;
GRANT SELECT ON TABLE raw_data.session_transaction TO GROUP analytics_users;
GRANT SELECT ON TABLE raw_data.channel TO GROUP analytics_users;
```
