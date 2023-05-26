{
    "job": {
        "setting": {
            "speed": {
                "channel": 1
            }
        },
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                       "username":"root",
	       "password":"123",
                        "column": [
                               "id","activity_name","activity_type","activity_desc","start_time","end_time","create_time"
                        ],
                        "fileType": "text",
                        "encoding": "UTF-8",
                        "fieldDelimiter": "\t"
	        "nullFormat":"\\N"
                    }
                },
                "writer": {
                    "name": "mysqlwriter",
                    "parameter": {
                        "writeMode": "replace",
                        "username": "root",
                        "password": "fuxiaoluo",
                        "column": [
               "id","name","region_id","area_code","iso_code","iso_3166_2"
                        ],
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:mysql://hadoop102:3306/gmall?useUnicode=true&characterEncoding=UTF-8",
                                "table": [
                                    "test_province"
                                ]
                            }
                        ]
                    }
                }
            }
        ]
    }
}