## --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## mysql执行方法一: pymysql连接池执行

```py

# coding: utf-8
import traceback
import pymysql


media_db = {
    'host': '192.168.23.176',
    'user': 'root',
    'password': '123456',
    'port': 3306,
    'charset': 'utf8',
    'database': 'homed_spider2'
}


class DbBaseQueue(object):
    """
    数据库连接池
    """

    # connect_info 为数据库连接参数
    def __init__(self, queue_max=10, **connect_info):
        import queue
        self.max = queue_max
        # 创建队列
        self._queue = queue.Queue(self.max)
        self._init_db(**connect_info)

        print("程序初始化完成", self._queue)

    def _conn(self):
        # 默认阻塞出队，且阻塞时间无限长
        conn = self._queue.get()
        # ping一下，确保还在连接
        conn.ping()
        return conn

    def conn(self):
        return self._conn()

    def _init_db(self, **params):
        for each in range(self.max):
            self._queue.put(pymysql.connect(**params))

    def _put(self, conn):
        self._queue.put(conn)

    def execute(self, sql: str, autocommit: int = 0):
        query = 0
        if sql.lower().startswith(("select", "desc")):  # 字符小写后检查是否以指定子字符串开头
            query = 1
        conn = self._conn()
        # 创建游标作用：让查询结果以列表嵌套字典格式输出
        # 使用cursor()方法创建一个游标对象cursor
        cursor = conn.cursor()
        return_flag = True
        try:
            # 使用execute()方法执行SQL
            res = cursor.execute(sql)
        except Exception as err:
            print('执行sql语句出错。sql={}, error={}'.format(sql, traceback.format_exc()))
            if autocommit:
                self.rollback(conn)
            return_flag = False
        else:
            if autocommit:
                self.commit(conn)
        finally:
            queryres = 0

            if query and return_flag:
                # 返回游标
                queryres = cursor

            cursor.close() if cursor else None
            if query:
                return queryres

            return return_flag

    def commit(self, conn):
        conn.commit()
        self._put(conn)

    def rollback(self, conn):
        conn.rollback()
        self._put(conn)

    def __enter__(self):
        conn = self._conn()

    def __exit__(self, exc_type, exc_val, exc_tb):
        pass

    def __del__(self):
        while True:
            conn = self._queue.get()
            conn.close()
            del conn
            if self._queue.empty():
                break


class MediaDbQueue(DbBaseQueue):

    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, "_instance"):
            cls._instance = object.__new__(cls)
        return cls._instance

    def __init__(self, queue_max):
        super(MediaDbQueue, self).__init__(queue_max, **media_db)


mediadbhandler = MediaDbQueue(2)


if __name__ == '__main__':
    start_time = '2023-07-01 00:00:00'
    end_time = '2023-07-25 00:00:00'
    video_upload_list = []
    sql = 'select requert_id, status, user, cloudpath, updatetime, cuttotal, video_id, name, is_timeout, f_domain as text from video_upload where updatetime >= "{}" and updatetime <= "{}"'.format(start_time, end_time)

    cursor = mediadbhandler.execute(sql, autocommit=1)
    database = cursor.fetchall()
    # 返回数据类型 元组中嵌套元组   eg: ((), (), ())
    if database:
        for each in database:
            ret_msg = {
                'video_hash': each[0],
                'status': each[1],
                'user': each[2],
                'cloudpath': each[3],
                'updatetime': each[4],
                'cuttotal': each[5],
                'video_id': each[6],
                'name': each[7],
                'is_timeout': each[8],
                'f_domain': each[9]
            }
            video_upload_list.append(ret_msg)
            video_upload_list = video_upload_list

    print('返回字符列表。video_upload_list={}'.format(video_upload_list))

```


## --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## mysql执行方法二: pymysql执行

```py

#  -*- coding: utf-8 -*-

import pymysql


class DbOperate:

    def __init__(self, host, user, password, dbname, port=3306):
        self.dbHandler = pymysql.connect(db=dbname, user=user, password=password, host=host, port=3306, charset='utf8')

    def __del__(self):
        if self.dbHandler:
            self.dbHandler.close()

    def operation(self, strSql):

        cursor = None
        nAffectLine = 0
        dataTuple = ()

        try:
            # 重连机制
            self.dbHandler.ping()
            cursor = self.dbHandler.cursor()
            nAffectLine = cursor.execute(strSql)
            self.dbHandler.commit()

            if nAffectLine > 0:
                dataTuple = cursor.fetchall()

        except Exception as err:
            self.dbHandler.rollback()
            return -1, err, None

        finally:
            if cursor:
                cursor.close()

        return nAffectLine, "", dataTuple

    def close(self):

        self.dbHandler.close()
        self.dbHandler = None


db_dict = {
    'host': '192.168.106.215',
    'user': 'homed',
    'password': 'newclustersql@dev210',
    'dbname': 'homed_dtvs'
}

run = DbOperate(**db_dict)


def task():

    SQL_add_series = 'SELECT t1.series_num, t1.actors, t1.director, t1.series_year, ' \
                     't1.country, t1.f_score ' \
                     'FROM video_series as t1 WHERE t1.series_id = 300006203;'

    SQL_add_video = 'SELECT t2.f_series_index, t3.record_user, t4.f_resolution, t5.language ' \
                    'FROM t_video_index as t2 LEFT JOIN video_record as t3 ON (t2.f_video_id = t3.video_id and t3.record_status=10000) ' \
                    'LEFT JOIN t_video_stream_info as t4 ON (t2.f_video_id = t4.f_video_id and t4.f_rate_level = "org") ' \
                    'LEFT JOIN video_info as t5 ON (t2.f_video_id = t5.video_id) ' \
                    'WHERE t2.f_video_id = 100124294;'

    SQL_INNER = 'SELECT t1.video_name, t1.status, t4.f_partner_english_name, ' \
                't4.f_partner_name, t3.series_name, t3.labels, t1.video_time as mov_duration ' \
                'FROM video_info as t1 INNER JOIN t_video_index as t2 ON (t1.video_id = t2.f_video_id) ' \
                'INNER JOIN video_series as t3 ON (t3.series_id = t2.f_series_id) ' \
                'INNER JOIN t_bussiness_partner as t4 ON (t4.f_partner_english_name = t1.f_cp_id) ' \
                'WHERE t1.video_id = 100124294 AND t2.f_series_id = 300006203;'

    SQL_LEFT = 'SELECT t1.video_name, t1.status, t4.f_partner_english_name, ' \
               't4.f_partner_name, t3.series_name, t3.labels, t1.video_time as mov_duration ' \
               'FROM video_info as t1 LEFT JOIN t_video_index as t2 ON (t1.video_id = t2.f_video_id) ' \
               'LEFT JOIN video_series as t3 ON (t3.series_id = t2.f_series_id) ' \
               'LEFT JOIN t_bussiness_partner as t4 ON (t4.f_partner_english_name = t1.f_cp_id) ' \
               'WHERE t1.video_id = 100124294 AND t2.f_series_id = 300006203;'

    ret_series, msg_series, data_series = run.operation(strSql=SQL_add_series)
    ret_video, msg_video, data_video = run.operation(strSql=SQL_add_video)
    ret_INNER, msg_INNER, data_INNER = run.operation(strSql=SQL_INNER)
    ret_LEFT, msg_LEFT, data_LEFT = run.operation(strSql=SQL_LEFT)

    print('返回数据。database={}'.format(data_series))
    # 数据类型 元组嵌套元组  eg: ((), (), ())


if __name__ == '__main__':
    task()

```


## --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## mysql执行方法三: peewee数据库执行

```py

#  -*- coding: utf-8 -*-

from peewee import *
from peewee import __exception_wrapper__

# 筛减掉第三方的日志打印
logging.getLogger('peewee').setLevel(logging.WARNING)


#捕获数据库连接八小时断开异常
class RetryOperationalError(object):
    def execute_sql(self, sql, params=None, commit=True):
        try:
            cursor = super(RetryOperationalError, self).execute_sql(sql, params, commit)
        except OperationalError:
            if not self.is_closed():
                self.close()
            with __exception_wrapper__:
                cursor = self.cursor()
                cursor.execute(sql, params or ())
                if commit and not self.in_transaction():
                    self.commit()
        return cursor


class RetryMySQLDatabase(RetryOperationalError, MySQLDatabase):
    pass


database = RetryMySQLDatabase('homed_dtvs', host='192.168.23.176', port=3306, user='root', password='123456', charset='utf8')


def ping():
    """重连机制，外部周期调用"""
    global database
    try:
        database.get_conn().ping()
    except Exception, e:
        if e.args[0] == 2006:
            if not database.is_closed():
                database.close()
            database.connect()
        else:
            #in peewee3.x, get_conn() has changed to Database.connection()
            if "no attribute 'get_conn'" in e.message:
                if database.connection():
                    return
            raise


# 定义一个基类，其他数据表的ORM继承之
class BaseModel(Model):
    class Meta:
        database = database


class DBTExportAsset(BaseModel):   #peewee在创建自己的field，可以直接操作数据库中的数据
    """分发媒资，资产"""
    f_asset_id      = IntegerField(null=True)      ##资产id
    f_long_asset_id = CharField(null=True)         ##长资产id
    f_status        = IntegerField(null=True)      ##状态
    f_task_id       = CharField(index=True)        ##任务ID
    f_update_time   = DateTimeField(null=True)     ##更新时间
    f_extern_data   = CharField(null=True)         ##附加数据
    f_asset_name    = CharField(index=True)        ##资产名字
    f_provider_id   = CharField(index=True)        ##提供商ID
    f_content_type  = IntegerField(null=True)      ##内容类型
    f_project_code  = IntegerField(null=True)      ##项目编码
    f_export_code   = IntegerField(null=True)      ##分发对象编码
    f_start_time    = DateTimeField(null=True)     ##媒资分发开始时间
    f_series_id     = CharField(null=True)         ##资产剧集长id

    # 元类，固定写法
    class Meta:
        # 数据表
        db_table = 't_export_asset_v2'
        # 定义索引
        # 第二部分是一个布尔值，指示索引是否应该唯一
        indexes = (
            (('f_asset_id', 'f_project_code', 'f_export_code'), True),
        )
        # 定义主键
        primary_key = CompositeKey('f_asset_id', 'f_export_code', 'f_project_code')


# 创建数据库表
def create_tables():
    database.connect()
    database.create_tables([DBTExportAsset])



# def is_exportseries(exportObjcode):  # 剧集存在分发记录
#     return len(DBTExportAsset.select().where(DBTExportAsset.f_series_id == nSeriesId, DBTExportAsset.f_export_code == exportObjcode))

if __name__ == '__main__':
    create_tables()
    print('执行更新')
    qry = DBTExportAsset.update(f_status=99).where((DBTExportAsset.f_asset_id == 100000030)&(DBTExportAsset.f_project_code == "62")&(DBTExportAsset.f_export_code == "1"))
    qry = DBTExportAsset.update(f_status=99).where(DBTExportAsset.f_asset_id == 100000030, DBTExportAsset.f_project_code == "62", DBTExportAsset.f_export_code == "1")
    print(qry.sql())
    qry.execute()

```


## --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## mysql执行方法四: pymysql数据库执行

```py

#  -*- coding: utf-8 -*-

import pymysql

mysqlSrv_ip = '192.168.53.18'
mysqlUser = 'homed'
mysqlPassword = 'Bj0erb4FQ87OA48N330WA0uHGh0='
mysqlDB = 'homed_dtvs'


# 执行数据库获取数据
def mysql(sql):
    db = pymysql.connect(host = mysqlSrv_ip, port = 3306, user = mysqlUser, password = mysqlPassword, database = mysqlDB, charset = 'utf8')
    cursor = db.cursor()

    # 字典化
    cursor = db.cursor(cursor=pymysql.cursors.DictCursor)

    # 执行sql语句
    cursor.execute(sql)

    # 获取SQL语句执行返回数据
    data = cursor.fetchall()

    # 将sql语句执行结果保存至数据库
    db.commit()

    # 关闭数据库连接
    db.close()
    return data


if __name__ == '__main__':
    series_list = []

    sql_14 = 'select series_id from video_series where f_cp_id = "iCNTV"'
    data_series_list = mysql(sql_14)

    # 返回字典格式数据
    for each in data_series_list:
        series_list.append(each['series_id'])

```
