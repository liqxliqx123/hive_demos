### 创建t_rating
```mysql
create external table t_rating_liqixuan(userid int, movieid int, rate int, times int
) ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe' 
WITH SERDEPROPERTIES ("field.delim" = "::") location '/data/hive/ratings.dat';
load data local inpath '/data/hive/ratings.dat' into table default.t_rating_liqixuan;
```


### 创建t_movie
```mysql
CREATE external TABLE  t_movie_liqixuan(
MovieID STRING, MovieName STRING, MovieType STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe'
WITH SERDEPROPERTIES ("field.delim" = "::")
LOCATION '/data/hive/movies.dat'; 
load data local inpath '/data/hive/movies.dat' into table default.t_movie_liqixuan;
```

### 创建t_user
```mysql
CREATE external TABLE t_user_liqixuan(userid int,sex string, age int, occupation int, zipcode int) ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe' WITH SERDEPROPERTIES ("field.delim" = "::") LOCATION '/data/hive/users.dat'; 
load data local inpath '/data/hive/users.dat' into table default.t_user_liqixuan;
```



### 题目1
展示电影 ID 为 2116 这部电影各年龄段的平均影评分
```mysql
select u.age, avg(rate) from t_movie_liqixuan m,t_user_liqixuan u,t_rating_liqixuan r where m.movieid = r.movieid and u.userid = r.userid and m.movieid=2116  group by u.age;
```

### 题目2
找出男性评分最高且评分次数超过 50 次的 10 部电影，展示电影名，平均影评分和评分次数
```mysql
select u.sex,m.moviename, avg(rate) as avg_rate, count(rate) as rate_cnt from t_movie_liqixuan m,t_user_liqixuan u,t_rating_liqixuan r where m.movieid = r.movieid and u.userid = r.userid and u.sex='M' group by m.moviename, u.sex having rate_cnt > 50 order by avg_rate desc limit 10;
```

### 题目3
```mysql
select m.moviename, avg(rate) as avg_rate from t_movie_liqixuan m,t_user_liqixuan u,t_rating_liqixuan r where m.movieid = r.movieid and u.userid = r.userid and m.moviename in 
(select t.moviename from (select m.moviename from t_movie_liqixuan m,t_user_liqixuan u,t_rating_liqixuan r where m.movieid = r.movieid and u.userid = r.userid and u.userid  in 
(select t.userid from (select u.userid, count(r.rate) as rate_cnt from t_movie_liqixuan m,t_user_liqixuan u,t_rating_liqixuan r where m.movieid = r.movieid and u.userid = r.userid and u.sex='F' group by u.userid order by rate_cnt desc limit 1) as t) 
order by r.rate desc limit 10) as t) group by m.moviename;
```
