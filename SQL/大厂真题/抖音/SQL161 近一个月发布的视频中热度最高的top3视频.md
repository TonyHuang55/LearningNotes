# SQL161 近一个月发布的视频中热度最高的top3视频

## 描述

现有用户-视频互动表tb_user_video_log

| id   | uid  | video_id | start_time          | end_time            | if_follow | if_like | if_retweet | comment_id |
| ---- | ---- | -------- | ------------------- | ------------------- | --------- | ------- | ---------- | ---------- |
| 1    | 101  | 2001     | 2021-09-24 10:00:00 | 2021-09-24 10:00:30 | 1         | 1       | 1          | NULL       |
| 2    | 101  | 2001     | 2021-10-01 10:00:00 | 2021-10-01 10:00:31 | 1         | 1       | 0          | NULL       |
| 3    | 102  | 2001     | 2021-10-01 10:00:00 | 2021-10-01 10:00:35 | 0         | 0       | 1          | NULL       |
| 4    | 103  | 2001     | 2021-10-03 11:00:50 | 2021-10-03 10:00:35 | 1         | 1       | 0          | 1732526    |
| 5    | 106  | 2002     | 2021-10-02 11:00:05 | 2021-10-02 11:01:04 | 2         | 0       | 1          | NULL       |
| 6    | 107  | 2002     | 2021-10-02 10:59:05 | 2021-10-02 11:00:06 | 1         | 0       | 0          | NULL       |
| 7    | 108  | 2002     | 2021-10-02 10:59:05 | 2021-10-02 11:00:05 | 1         | 1       | 1          | NULL       |
| 8    | 109  | 2002     | 2021-10-03 10:59:05 | 2021-10-03 11:00:01 | 0         | 1       | 0          | NULL       |
| 9    | 105  | 2002     | 2021-09-25 11:00:00 | 2021-09-25 11:00:30 | 1         | 0       | 1          | NULL       |
| 10   | 101  | 2003     | 2021-09-26 11:00:00 | 2021-09-26 11:00:30 | 1         | 0       | 0          | NULL       |
| 11   | 101  | 2003     | 2021-09-30 11:00:00 | 2021-09-30 11:00:30 | 1         | 1       | 0          | NULL       |

（uid-用户ID, video_id-视频ID, start_time-开始观看时间, end_time-结束观看时间, if_follow-是否关注, if_like-是否点赞, if_retweet-是否转发, comment_id-评论ID）

短视频信息表tb_video_info

| id   | video_id | author | tag  | duration | release_time        |
| ---- | -------- | ------ | ---- | -------- | ------------------- |
| 1    | 2001     | 901    | 旅游 | 30       | 2021-09-05 07:00:00 |
| 2    | 2002     | 901    | 旅游 | 60       | 2021-09-05 07:00:00 |
| 3    | 2003     | 902    | 影视 | 90       | 2021-09-05 07:00:00 |
| 4    | 2004     | 902    | 影视 | 90       | 2021-09-05 08:00:00 |

（video_id-视频ID, author-创作者ID, tag-类别标签, duration-视频时长, release_time-发布时间）

**问题**：找出近一个月发布的视频中热度最高的top3视频。

**注**：

- 热度=(a*视频完播率+b*点赞数+c*评论数+d*转发数)*新鲜度；
- 新鲜度=1/(最近无播放天数+1)；
- 当前配置的参数a,b,c,d分别为100、5、3、2。
- 最近播放日期以**end_time-结束观看时间**为准，假设为T，则最近一个月按[T-29, T]闭区间统计。
- 结果中热度保留为**整数**，并按热度**降序**排序。

**输出示例**：

示例数据的输出结果如下

| video_id | hot_index |
| -------- | --------- |
| 2001     | 122       |
| 2002     | 56        |
| 2003     | 1         |

解释：

最近播放日期为2021-10-03，记作当天日期；近一个月（2021-09-04及之后）发布的视频有2001、2002、2003、2004，不过2004暂时还没有播放记录；

视频2001完播率1.0（被播放次数4次，完成播放4次），被点赞3次，评论1次，转发2次，最近无播放天数为0，因此热度为：(100*1.0+5*3+3*1+2*2)/(0+1)=122

同理，视频2003完播率0，被点赞数1，评论和转发均为0，最近无播放天数为3，因此热度为：(100*0+5*1+3*0+2*0)/(3+1)=1（1.2保留为整数）。

## 示例1

输入：

```
DROP TABLE IF EXISTS tb_user_video_log, tb_video_info;
CREATE TABLE tb_user_video_log (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '自增ID',
    uid INT NOT NULL COMMENT '用户ID',
    video_id INT NOT NULL COMMENT '视频ID',
    start_time datetime COMMENT '开始观看时间',
    end_time datetime COMMENT '结束观看时间',
    if_follow TINYINT COMMENT '是否关注',
    if_like TINYINT COMMENT '是否点赞',
    if_retweet TINYINT COMMENT '是否转发',
    comment_id INT COMMENT '评论ID'
) CHARACTER SET utf8 COLLATE utf8_bin;

CREATE TABLE tb_video_info (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '自增ID',
    video_id INT UNIQUE NOT NULL COMMENT '视频ID',
    author INT NOT NULL COMMENT '创作者ID',
    tag VARCHAR(16) NOT NULL COMMENT '类别标签',
    duration INT NOT NULL COMMENT '视频时长(秒数)',
    release_time datetime NOT NULL COMMENT '发布时间'
)CHARACTER SET utf8 COLLATE utf8_bin;

INSERT INTO tb_user_video_log(uid, video_id, start_time, end_time, if_follow, if_like, if_retweet, comment_id) VALUES
   (101, 2001, '2021-09-24 10:00:00', '2021-09-24 10:00:30', 1, 1, 1, null)
  ,(101, 2001, '2021-10-01 10:00:00', '2021-10-01 10:00:31', 1, 1, 0, null)
  ,(102, 2001, '2021-10-01 10:00:00', '2021-10-01 10:00:35', 0, 0, 1, null)
  ,(103, 2001, '2021-10-03 11:00:50', '2021-10-03 11:01:35', 1, 1, 0, 1732526)
  ,(106, 2002, '2021-10-02 10:59:05', '2021-10-02 11:00:04', 2, 0, 1, null)
  ,(107, 2002, '2021-10-02 10:59:05', '2021-10-02 11:00:06', 1, 0, 0, null)
  ,(108, 2002, '2021-10-02 10:59:05', '2021-10-02 11:00:05', 1, 1, 1, null)
  ,(109, 2002, '2021-10-03 10:59:05', '2021-10-03 11:00:01', 0, 1, 0, null)
  ,(105, 2002, '2021-09-25 11:00:00', '2021-09-25 11:00:30', 1, 0, 1, null)
  ,(101, 2003, '2021-09-26 11:00:00', '2021-09-26 11:00:30', 1, 0, 0, null)
  ,(101, 2003, '2021-09-30 11:00:00', '2021-09-30 11:00:30', 1, 1, 0, null);

INSERT INTO tb_video_info(video_id, author, tag, duration, release_time) VALUES
   (2001, 901, '旅游', 30, '2021-09-05 7:00:00')
  ,(2002, 901, '旅游', 60, '2021-09-05 7:00:00')
  ,(2003, 902, '影视', 90, '2021-09-05 7:00:00')
  ,(2004, 902, '影视', 90, '2021-09-05 8:00:00');
```

输出：

```
2001|122
2002|56
2003|1
```

## 解析

```
SELECT
    t.video_id,
    round((100*a + 5*b + 3*c + 2*d)/(t.e+1), 0) AS hot_index
    FROM 
    (
    SELECT
        vi.video_id, 
        AVG(if(TIME_TO_SEC(TIMEDIFF(end_time,start_time)) >= duration, 1, 0)) AS a,
        SUM(if_like) AS b,
        COUNT(comment_id) AS c,
        SUM(if_retweet) AS d, 
        DATEDIFF((SELECT MAX(end_time) FROM tb_user_video_log), MAX(end_time)) AS e
        FROM tb_video_info vi INNER JOIN tb_user_video_log USING(video_id)
        WHERE DATEDIFF((SELECT MAX(end_time) from tb_user_video_log), release_time) <= 29
        GROUP BY video_id
    ) t
    ORDER BY hot_index DESC
    LIMIT 3
```