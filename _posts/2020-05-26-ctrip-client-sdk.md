---
layout: post
title: 携程酒店SDK封装
categories: [Encapsulation]
tags: [Encapsulation]
fullview: true
markdown: kramdown

---

最近做的分销业务，携程只提供了HTTP API，没有提供php sdk。这方面，淘宝做的不错（飞猪）直接点击下载sdk。

虽然提供了两个客户端sdk，跟现有框架融合是一个问题。淘宝的sdk直接利用`composer`包的方式加载:
```
╰─$ ls -lh .
total 744
-rw-r--r--   1 martin  staff   6.3K  5 23 16:48 README.md
....
-rw-r--r--   1 martin  staff   229B  5  8 12:02 codeception.yml
-rw-r--r--   1 martin  staff   3.0K  5 26 17:33 composer.json
-rw-r--r--   1 martin  staff   327K  5 26 17:33 composer.lock
-rw-r--r--   1 martin  staff    15K  5 26 17:33 symfony.lock
lrwxr-xr-x   1 martin  staff    43B  5  8 12:02 taobao-sdk -> taobao-sdk-PHP-auto_1568088312300-20191114/
drwxr-xr-x  11 martin  staff   352B  5  8 12:02 taobao-sdk-PHP-auto_1568088312300-20191114
drwxr-xr-x  53 martin  staff   1.7K  5 26 17:33 vendor
```

composer.json再加入仓库引入源:
```
    },
    "repositories": [
        {
            "type": "path",
            "url": "./taobao-sdk"
        }
    ],
    "minimum-stability":"dev"
}
```
这样就纳入了包管理。

淘宝的sdk，需要增加一个`composer.json`文件。
```
{
    "name": "martin/taobao-sdk",
    "authors": [
        {
            "name": "liujingyu",
            "email": "liujingyu@xiaozhu.com"
        }
    ],
    "require": {},
    "autoload":{
        "classmap": [
            "top/",
            "aliyun/",
            "dingtalk/",
            "QimenCloud/"
        ]
    },
    "minimum-stability":"dev"
}
```

安装可。
```
composer install martin/taobao-sdk
```

接下来聊聊，携程sdk`https://dev.ctrip.com/#/document?id=83`。
基于携程接口特点设计如下:
```php

class CtripConfig
{
    const SOURCE = CommonSetting::CTRIP;
    const MODE = 1;
    const FORMAT = 'json';
    const REFRESH_TOKEN_EXPIRE = 900; // expire 15min

    // 携程API列表
    const API_LIST = [
        // api_name
        // @see ....
        //'api_name' => [
            //'path' => [
                //'prod' => '',
                //'test' => '',
                //'dev' => '',
                //'*' => '',
            //],
        //],

        // 获取全量城市信息
        // @see https://dev.ctrip.com/#/document?id=121
        'getAllCityInfo' => [
            'path' => [
                'prod' => 'f321ca3356b643ec8e32db0aec6a4b70',
                'test' => 'da97859659134aab9bfea1afbfa96d83',
                'dev' => 'da97859659134aab9bfea1afbfa96d83',
                '*' => 'da97859659134aab9bfea1afbfa96d83',
            ],
            'rate' => 200, // 每分钟200次
        ],

        // 获取酒店清单
        // @see https://dev.ctrip.com/#/document?id=144
        'getHotelList' => [
            'path' => [
                'prod' => '3a46e1d3100d46d8a46f52c3dc6b0273',
                'test' => '4c81467f1e5843fbb0a5eaba23906d70',
                'dev' => '4c81467f1e5843fbb0a5eaba23906d70',
                '*' => '4c81467f1e5843fbb0a5eaba23906d70',
            ],
            'rate' => 200, // 每分钟200次
        ],

        // 获取酒店静态信息
        // @see https://dev.ctrip.com/#/document?id=145
        'getHotelStaticInfo' => [
            'path' => [
                'prod' => 'becf5b5008064a26be77cac065688ca7',
                'test' => '62a8bb573582435bbe8b361334efe36f',
                'dev' => '62a8bb573582435bbe8b361334efe36f',
                '*' => '62a8bb573582435bbe8b361334efe36f',
            ],
            'rate' => 600, //每分钟600
        ],

        // 获取房型静态信息
        // @see https://dev.ctrip.com/#/document?id=146
        'getRoomTypeStaticInfo' => [
            'path' => [
                'prod' => '39122604bf0e4b33a9569658cf273161',
                'test' => 'c04d1d7b7af449999dba88896fcc0811',
                'dev' => 'c04d1d7b7af449999dba88896fcc0811',
                '*' => 'c04d1d7b7af449999dba88896fcc0811',
            ],
            'rate' => 600, //每分钟600
        ],

        // 监测静态信息变化
        // @see https://dev.ctrip.com/#/document?id=56
        'listenStaticInfo' => [
            'path' => [
                'prod' => '66fd1e3089fb448e912808cccbe017c0',
                'test' => 'd0e65061aef24f88854031c60cabfd91',
                'dev' => 'd0e65061aef24f88854031c60cabfd91',
                '*' => 'd0e65061aef24f88854031c60cabfd91',
            ],
            'filter' => 'ChangeInfos',
            'rate' => 1000, // 每分钟1000次
        ],

        // 监测房价、房量、房态增量变化接口
        // @see https://dev.ctrip.com/#/document?id=98
        'listenRoomPriceNumberStatus' => [
            'path' => [
                'prod' => 'a8f721b9716a41ca9df8b326f10655df',
                'test' => '68c959f3d5a94d7ca4ddd97cbe97d424',
                'dev' => '68c959f3d5a94d7ca4ddd97cbe97d424',
                '*' => '68c959f3d5a94d7ca4ddd97cbe97d424',
            ],
            'filter' => 'ChangeInfos',
            'rate' => 1000, // 每分钟1000次
        ],

        // 监测房型上下线
        // @see https://dev.ctrip.com/#/document?id=161
        'listenRoomTypeOnAndOffLine' => [
            'path' => [
                'prod' => '4536a4178961443a937423c1eb8fa2ba',
                'test' => 'ef89b1f6ec8e49d39d5debac065a9a4a',
                'dev' => 'ef89b1f6ec8e49d39d5debac065a9a4a',
                '*' => 'ef89b1f6ec8e49d39d5debac065a9a4a',
            ],
            'filter' => 'RoomStatusChangeItems',
            'rate' => 100, // 每分钟100次
        ],

        // 每日价格拉取接口（仅国内酒店）<落地价>
        // @see https://dev.ctrip.com/#/document?id=95
        'fetchLandPrice' => [
            'path' => [
                'prod' => '',
                'test' => '99503c56b9fa4417a8bc77b92ea56d41',
                'dev' => '99503c56b9fa4417a8bc77b92ea56d41',
                '*' => '99503c56b9fa4417a8bc77b92ea56d41',
            ],
        ],

        // 报价实时查询接口（国内酒店+海外酒店） <直连价>
        // @see https://dev.ctrip.com/#/document?id=94
        'fetchDirectPrice' => [
            'path' => [
                'prod' => '4c15cadb05f14aa9836260c53ab74526',
                'test' => '14a14763194648f5b9d2427f00b3ca04',
                'dev' => '14a14763194648f5b9d2427f00b3ca04',
                '*' => '14a14763194648f5b9d2427f00b3ca04',
            ],
            'rate' => 6000, //每分钟6000次
        ],

        // 可预订检查
        // @see https://dev.ctrip.com/#/document?id=62
        'checkBookAble' => [
            'path' => [
                'prod' => '419f7a688e3c45f481d295708b870323',
                'test' => 'd67d24cfdc2940cd858f506c708cf0d4',
                'dev' => 'd67d24cfdc2940cd858f506c708cf0d4',
                '*' => 'd67d24cfdc2940cd858f506c708cf0d4',
            ]
        ],

        // 订单保存
        // @see https://dev.ctrip.com/#/document?id=63
        'saveOrderBook' => [
            'path' => [
                'prod' => '2b0c7e0eebe2467a97be3d284313a129',
                'test' => 'acf5c44749234668ba3c539cc1d26c87',
                'dev' => 'acf5c44749234668ba3c539cc1d26c87',
                '*' => 'acf5c44749234668ba3c539cc1d26c87',
            ]
        ],

        // 订单提交
        // @see https://dev.ctrip.com/#/document?id=64
        'submitOrderBook' => [
            'path' => [
                'prod' => 'f40c5c71f84745cbbc73238252d9d43c',
                'test' => '5f3c81bc0c90458ba94bfe2dafea8459',
                'dev'  => '5f3c81bc0c90458ba94bfe2dafea8459',
                '*'    => '5f3c81bc0c90458ba94bfe2dafea8459',
            ]
        ],

        // 获取订单详情
        // @see https://dev.ctrip.com/#/document?id=67
        'getOrderBookDetail' => [
            'path' => [
                'prod' => 'e4a2a8ac057f4b378c9e153ef3f35249',
                'test' => '5db9d7e805ef4b36a8cf0e57900e118a',
                'dev'  => '5db9d7e805ef4b36a8cf0e57900e118a',
                '*'    => '5db9d7e805ef4b36a8cf0e57900e118a',
            ]
        ],

        // 取消订单
        // @see https://dev.ctrip.com/#/document?id=68
        'cancelBookOrder' => [
            'path' => [
                'prod' => 'f40a36ac2dfe42518fb883d33fa33391',
                'test' => '519114d791b94babba31f32eb5d559eb',
                'dev'  => '519114d791b94babba31f32eb5d559eb',
                '*'    => '519114d791b94babba31f32eb5d559eb',
            ]
        ],

        // 检测订单状态变化
        // @see https://dev.ctrip.com/#/document?id=66
        'listenOrderStatusChange' => [
            'path' => [
                'prod' => 'e5e981e2cf9340f99f9cbf79c82668ac',
                'test' => '61aebb32ae1347c5a274fc9fb4b4f962',
                'dev'  => '61aebb32ae1347c5a274fc9fb4b4f962',
                '*'    => '61aebb32ae1347c5a274fc9fb4b4f962',
            ]
        ],

        //查询某城市下各酒店的起价
        // @see https://dev.ctrip.com/#/document?id=93
        'fetchLowerPrice' => [
            'path' => [
                'prod' => '4c6f5dcc4663443b8d8ae044ad8bee10',
                'test' => 'c659a717bf23422b907ff7f9c75022ed',
                'dev'  => 'c659a717bf23422b907ff7f9c75022ed',
                '*'    => 'c659a717bf23422b907ff7f9c75022ed',
            ],
            'rate' => 200, // 每分钟200次
        ]
    ];

    /**
     * 根据key获取携程api icode
     *
     * @param $api
     * @return bool|string
     */
    public static function getCtripIcode($api)
    {
        $env = getenv('APP_ENV');
        $default = '*';
        if (isset(CtripConfig::API_LIST[$api]['path'][$env])) {
            return CtripConfig::API_LIST[$api]['path'][$env];
        } else {
            if (isset(CtripConfig::API_LIST[$api])) {
                return CtripConfig::API_LIST[$api]['path'][$default];
            } else {
                return false;
            }
        }
    }

    /**
     * 获取结果过滤器
     */
    public static function getApiFilter($api)
    {
        if (empty(CtripConfig::API_LIST[$api]['filter'])) {
            return false;
        }

        return CtripConfig::API_LIST[$api]['filter'];
    }
}

class CtripToken
{

    private $aid;
    private $sid;
    private $key;
    private $container;
    private $redis;
    private $client;
    private $logger;

    public function __construct(
        ContainerInterface $container,
        ClientInterface $redis,
        LoggerInterface $logger
    ) {
        $this->container = $container;
        $this->redis = $redis;
        $this->client = $this->container->get('bl_guzzle.client.ctrip');
        $this->logger = $logger;

        $this->aid = getenv('CTRIP_AID');
        $this->sid = getenv('CTRIP_SID');
        $this->key = getenv('CTRIP_KEY');
    }


    /**
     * (支持分布式获取Token)获取 token.
     */
    protected function getToken()
    {
        $token = $this->redis->get(self::SOURCE."_token");
        $refreshToken = $this->redis->get(self::SOURCE."_refresh_token");

        if (!empty($token)) {
            return $token;
        }

        // 加锁，避免重复刷新token
        $lock = function ($key) {
            return $this->redis->set("ctrip:lock", $key, "nx", "ex", 10);
        };

        $unlock = function ($key) {
            $script = <<<LUA
if redis.call("get",KEYS[1]) == ARGV[1]
then
    return redis.call("del",KEYS[1])
else
    return 0
end
LUA;
            return $this->redis->eval($script, 1, "ctrip:lock", $key);
        };

        $key = rand(1, 100000);

        token:
        if ($lock($key)) {
            if (empty($refreshToken)) {
                // token 初始化
                $res = $this->client->get(
                    '/openserviceauth/authorize.ashx?' .
                    http_build_query([
                        'AID' => $this->aid,
                        'SID' => $this->sid,
                        'KEY' => $this->key,
                    ])
                );
            } else {
                // token 更新
                $res = $this->client->get(
                    '/openserviceauth/refresh.ashx?'.
                    http_build_query([
                        'AID' => $this->aid,
                        'SID' => $this->sid,
                        'refresh_token' => $refreshToken,
                    ])
                );
            }

            // 解锁失败，说明锁超时，返回null，上层进行重试
            if (!$unlock($key)) {
                $this->logger->warning('unlock failure');
                return null;
            }
        } else {
            $this->logger->warning('lock failure');
            // 未取得锁，上层进行重试
            return null;
        }
        // 加锁结束

        if (200 != $res->getStatusCode()) {
            throw new Exception('CTRIP GET Token Http Client error');
        }

        $data = json_decode($res->getBody(), true);
        if (!empty($data['ErrCode']) && $data['ErrCode'] == 1005) {
            // 重置刷新Token
            $refreshToken = null;
            goto token;
        }
        if (empty($data['Expires_In']) || empty($data['Access_Token']) || empty($data['Refresh_Token'])) {
            throw new Exception('CTRIP GET TOKNE Http Client error, response data format error:'.json_encode($data));
        }

        $this->redis->setex(self::SOURCE."_token", $data['Expires_In'], $data['Access_Token']);
        $this->redis->setex(self::SOURCE."_refresh_token", self::REFRESH_TOKEN_EXPIRE, $data['Refresh_Token']);

        return $data['Access_Token'];
    }

    protected function removeCtripAccessToken()
    {
        return $this->redis->del(self::SOURCE."_token");
    }
}

class CtripClient
{

    private $aid;
    private $sid;
    private $key;
    private $container;
    private $redis;
    private $client;
    private $messageControl;
    private $logger;
    private $apiRateControl;

    public function __construct(
        ContainerInterface $container,
        ClientInterface $redis,
        MessageControl $messageControl,
        LoggerInterface $logger,
        APIRateControl $apiRateControl,
        Token $token
    ) {
        $this->container = $container;
        $this->redis = $redis;
        $this->client = $this->container->get('bl_guzzle.client.ctrip');
        $this->messageControl = $messageControl;
        $this->logger = $logger;
        $this->apiRateControl = $apiRateControl;
        $this->token = $token;
        $this->aid = getenv('CTRIP_AID');
        $this->sid = getenv('CTRIP_SID');
        $this->key = getenv('CTRIP_KEY');
    }

    /**
     * 支持 API_LIST 里面的所有接口.
     *
     * @param $name
     * @param $arguments
     * @return array
     * @throws FatalException
     */
    public function __call($name, $arguments)
    {
        if (isset(CtripConfig::API_LIST[$name])) {
            if ($arguments[0] instanceof HotelRequest) {
                return $this->getDetailData($name, $arguments[0]->toArray());
            }
            throw new \InvalidArgumentException();
        }

        throw new \BadMethodCallException();
    }

    /**
     * @param $response
     * @return array|mixed
     */
    protected function getContent($response)
    {
        if (200 == $response->getStatusCode()) {
            return \GuzzleHttp\json_decode($response->getBody(), true);
        } else {
            return [];
        }
    }

    /**
     * 获取数据接口.
     *
     * @param string $api
     * @param array $dataParam
     *
     * @return array
     **/
    protected function getDetailData($api, $dataParam)
    {
        try {
            if (!($icode = CtripConfig::getCtripIcode($api))) {
                throw new \InvalidArgumentException(sprintf("icode find by api, not found the api:%s", $api));
            }

            // api 接口测试限制
            if (($coolingSecond = $this->apiRateControl->getCoolingSecondForAPI($api))) {
                sleep($coolingSecond);
            }

token:
            $token = Util::try(3, function () {
                    return $this->token->getToken();
                    }, true);

            if (null == $token) {
                throw new \Exception('get token retry 3 failure');
            }

            $params = http_build_query([
                    'AID'   => $this->aid,
                    'SID'   => $this->sid,
                    'ICODE' => $icode,
                    'UUID'  => Util::getUUID(),
                    'Token' => $token,
                    'Mode'  => self::MODE,
                    'Format'=> self::FORMAT
            ]);

            $response = $this->client->post(
                    '/openservice/serviceproxy.ashx?'.$params,
                    [
                    'json' => $dataParam
                    ]
                    );

            $content = $this->getContent($response);
            if (!empty($content['ErrCode']) && $content['ErrCode'] == 232) {
                // clear store redis token and retry
                $this->token->removeCtripAccessToken();
                goto token;
            }

            if (($field = CtripConfig::getApiFilter($api)) && !empty($content[$field])) {
                // 去除重复调用的数据
                $content[$field] = $this->messageControl->filterMessage($content[$field]);
                return $content;
            } else {
                return $content;
            }
        } catch (Throwable $e) {
            $this->logger->error('CtripClient Throwable', [
                    'api' => $api,
                    'data' => $dataParam,
                    'message' => $e->getMessage(),
            ]);
        }

        return null;
    }

    public static function successAndReturnLastRecordId($response)
    {
        $successAndReturnLastRecordId = function ($response) {
            if (self::success($response)) {
                if (!empty($response['PagingInfo']['LastRecordID'])) {
                    return $response['PagingInfo']['LastRecordID'];
                }
            }
            return false;
        };
        return $successAndReturnLastRecordId($response);
    }

    public static function success($response)
    {
        $success = function ($response) {
            if (!empty($response['ResponseStatus']['Ack']) && $response['ResponseStatus']['Ack'] == 'Success') {
                return true;
            }
            return false;
        };
        return $success($response);
    }
}

// 工具类封装
class Util
{
	// 重试逻辑，正常逻辑和异常逻辑分开
	public static function retry($retries, callable $fn, $sleep = false)
    {
        $base = $retries * 2;

        beginning:
        try {
            return $fn();
        } catch (\Exception $e) {
            if (!$retries) {
                return null;
            }

            if ($sleep) {
                // 休眠时间递增(单位s), 1, 2, 5
                $sleepTime = ($base - $retries) / $retries;
                sleep($sleepTime);
            }

            $retries--;
            goto beginning;
        }
    }

    const LOOP_RUN_CONTINUE = 0;
    const LOOP_NORMAL_STOP = 1;
    const LOOP_API_EXCEPTION_STOP = 2;

    /**
     * 任务循环器, 可复用,用于分页数据获取.
     *
     * @param $run
     * @param mixed ...$args
     */
    public static function loop($run, ...$args)
    {
        $loop = function () use ($run, $args) {
            do {
                // status: 0 循环, 1 正常结束, 2 接口异常结束
                $status = $run(...$args);
                if ($status != 0) {
                    break;
                }
            } while (true);
        };
        $loop();
    }
}

// 携程接口API调用频次记录，并且返回冷却时间
class APIRateControl
{
    private $redis;

    const KEY_PREFIX = 'ctrip:rate:';

    public function __construct(ClientInterface $redis)
    {
        $this->redis = $redis;
    }

    /**
     * 获取API冷却时间.
     *
     * @param string $api
     * @param string $second
     * @param int $times
     *
     * @return int
     * @throws \Exception
     */
    public function getAPICoolingSecond($api, $second, $times)
    {
        $lockedTimes = function ($api, $expire, $times) {
            $script = <<<LUA
local times = redis.call('incr',KEYS[1])

-- 第一次访问的时候加上过期时间60秒（60秒过后从新计数）
if times == 1 then
    redis.call('expire',KEYS[1], ARGV[1])
end

-- 注意，从redis进来的默认为字符串，lua同种数据类型只能和同种数据类型比较
if times > tonumber(ARGV[2]) then
    return 0
end
return 1
LUA;
            return $this->redis->eval($script, 1, self::KEY_PREFIX.$api, $expire, $times);
        };

        if ($lockedTimes($api, $second, $times)) {
            return 0;
        }

        $sleep = $this->redis->ttl(self::KEY_PREFIX.$api);
        if ($sleep > 0) {
            return $sleep;
        } else {
            return 0;
        }
    }

    /**
     * 冷却时间.
     *
     * @param string $api
     *
     * @return int
     */
    public function getCoolingSecondForAPI($api)
    {
        if (empty(CtripConfig::API_LIST[$api]['rate'])) {
            return 0;
        }

        return $this->getAPICoolingSecond(
            $api,
            60,
            CtripConfig::API_LIST[$api]['rate']
        );
    }
}


<?php

use Adbar\Dot;

/**
 *
 * 使用如下：
 *  // 测试HotelRequest toArray
 * $r = new HotelRequest();
 * $r->PagingSettings_LastRecordID = 100;
 * $r->PagingSettings_PageSize = 200;
 * $r->Settings_ExtendedNodes = [ 'HotelStaticInfo.GeoInfo' ];
 * $r->Settings_PrimaryLangID = 'zh_cn';
 * echo $r->toJson(),PHP_EOL;
 * var_dump($r->toArray());
 */

class HotelRequest
{
    // 格式     'SearchCandidate_HotelID' => 'SearchCandidate.HotelID',
    const RULES = [
        // 酒店
        'SearchCandidate_HotelID' => 'SearchCandidate.HotelID',
        'SearchCandidate_RoomIDs' => 'SearchCandidate.RoomIDs',
        'Settings_PrimaryLangID' => 'Settings.PrimaryLangID',
        'Settings_ExtendedNodes' => 'Settings.ExtendedNodes',
        'Settings_SearchTags' => 'Settings.SearchTags',
        'PagingSettings_PageSize' => 'PagingSettings.PageSize',
        'PagingSettings_LastRecordID' => 'PagingSettings.LastRecordID',
        'SearchCandidate_DateTimeRange_Start' => 'SearchCandidate.DateTimeRange.Start',
        'SearchCandidate_DateTimeRange_End' => 'SearchCandidate.DateTimeRange.End',
        'SearchCandidate_StartTime' => 'SearchCandidate.StartTime',
        'SearchCandidate_Duration' => 'SearchCandidate.Duration',
        'Settings_ShowDataRange' => 'Settings.ShowDataRange',
        'Settings_IsShowChangeDetails' => 'Settings.IsShowChangeDetails',
        'Settings_DataCategory' => 'Settings.DataCategory',
        'SearchCandidate_SearchByType_SearchType' => 'SearchCandidate.SearchByType.SearchType',
        'SearchCandidate_SearchByType_IsHaveHotel' => 'SearchCandidate.SearchByType.IsHaveHotel',
        'SearchCandidate_SearchByCityID_CityID' => 'SearchCandidate.SearchByCityID.CityID',

        // 订单部分 //
        // 可预订检查
        'AvailRequestSegments_Version' => 'AvailRequestSegments.Version',
        'AvailRequestSegments_TransactionIdentifier' => 'AvailRequestSegments.TransactionIdentifier',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_HotelRef_HotelCode' =>
        'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.HotelRef.HotelCode',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_StayDateRange_Start' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.StayDateRange.Start',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_StayDateRange_End' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.StayDateRange.End',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RoomID' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RatePlanCandidates.RatePlanCandidate.RoomID',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RatePlanID' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RatePlanCandidates.RatePlanCandidate.RatePlanID',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RatePlanCategory' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RatePlanCandidates.RatePlanCandidate.RatePlanCategory',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_PrepaidIndicator' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RatePlanCandidates.RatePlanCandidate.PrepaidIndicator',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RoomStayCandidates_RoomStayCandidate_GuestCounts_GuestCount_Count' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RoomStayCandidates.RoomStayCandidate.GuestCounts.GuestCount.Count',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RoomStayCandidates_RoomStayCandidate_Quantity' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.RoomStayCandidates.RoomStayCandidate.Quantity',
        'AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_TPA_Extensions_LateArrivalTime' => 'AvailRequestSegments.AvailRequestSegment.HotelSearchCriteria.Criterion.TPA_Extensions.LateArrivalTime',
        // 保存订单
        'Version' => 'Version',
        'TransactionIdentifier' => 'TransactionIdentifier',
        'UniqueID' => 'UniqueID',
        'UniqueID_Type' => 'UniqueID.Type',
        'UniqueID_ID' => 'UniqueID.ID',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_RoomTypes_RoomType_NumberOfUnits' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.RoomTypes.RoomType.NumberOfUnits',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RoomID' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.RatePlans.RatePlan.RoomID',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RatePlanCategory' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.RatePlans.RatePlan.RatePlanCategory',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_PrepaidIndicator' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.RatePlans.RatePlan.PrepaidIndicator',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RatePlanID' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.RatePlans.RatePlan.RatePlanID',
        'HotelReservations_HotelReservation_RoomStays_RoomStay_BasicPropertyInfo_HotelCode' => 'HotelReservations.HotelReservation.RoomStays.RoomStay.BasicPropertyInfo.HotelCode',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_PersonName' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.PersonName',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_PersonName_Name' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.ContactPerson.PersonName.Name',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_Telephone_PhoneTechType' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.ContactPerson.Telephone.PhoneTechType',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_Telephone_PhoneNumber' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.ContactPerson.Telephone.PhoneNumber',

        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_ContactType' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.ContactPerson.ContactType',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.Profiles.ProfileInfo.Profile.Customer.ContactPerson',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_TPA_Extensions_LateArrivalTime' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.TPA_Extensions.LateArrivalTime',
        'HotelReservations_HotelReservation_ResGuests_ResGuest_ArrivalTime' => 'HotelReservations.HotelReservation.ResGuests.ResGuest.ArrivalTime',
        'HotelReservations_HotelReservation_ResGlobalInfo_GuestCounts_GuestCount_Count' => 'HotelReservations.HotelReservation.ResGlobalInfo.GuestCounts.GuestCount.Count',
        'HotelReservations_HotelReservation_ResGlobalInfo_TimeSpan_Start' => 'HotelReservations.HotelReservation.ResGlobalInfo.TimeSpan.Start',
        'HotelReservations_HotelReservation_ResGlobalInfo_TimeSpan_End' => 'HotelReservations.HotelReservation.ResGlobalInfo.TimeSpan.End',
        //'HotelReservations_HotelReservation_ResGlobalInfo_DepositPayments' => 'HotelReservations.HotelReservation.ResGlobalInfo.DepositPayments',
        'HotelReservations_HotelReservation_ResGlobalInfo_Total_ExclusiveAmount' => 'HotelReservations.HotelReservation.ResGlobalInfo.Total.ExclusiveAmount',
        'HotelReservations_HotelReservation_ResGlobalInfo_Total_InclusiveAmount' => 'HotelReservations.HotelReservation.ResGlobalInfo.Total.InclusiveAmount',
        'HotelReservations_HotelReservation_ResGlobalInfo_Total_CurrencyCode' => 'HotelReservations.HotelReservation.ResGlobalInfo.Total.CurrencyCode',
        'HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_ExclusiveAmount' => 'HotelReservations.HotelReservation.ResGlobalInfo.TPA_Extensions.TotalCost.ExclusiveAmount',
        'HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_InclusiveAmount' => 'HotelReservations.HotelReservation.ResGlobalInfo.TPA_Extensions.TotalCost.InclusiveAmount',
        'HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_CurrencyCode' => 'HotelReservations.HotelReservation.ResGlobalInfo.TPA_Extensions.TotalCost.CurrencyCode',

        // 订单提交
        'ReservationPayment_ReservationID_Type' => 'ReservationPayment.ReservationID.Type',
        'ReservationPayment_ReservationID_ID' => 'ReservationPayment.ReservationID.ID',
        'ReservationPayment_PaymentDetail' => 'ReservationPayment.PaymentDetail',
        'ReservationPayment_Invoice' => 'ReservationPayment.Invoice',

        // 监测订单状态变化
        'HotelReservations' => 'HotelReservations',
        'SearchTypeEntity_SearchType' => 'SearchTypeEntity.SearchType',
        'SearchTypeEntity_HotelCount' => 'SearchTypeEntity.HotelCount',
        'SearchTypeEntity_PageIndex' => 'SearchTypeEntity.PageIndex',
        'PublicSearchParameter_City' => 'PublicSearchParameter.City',
        'PublicSearchParameter_CheckInDate' => 'PublicSearchParameter.CheckInDate',
        'PublicSearchParameter_CheckOutDate' => 'PublicSearchParameter.CheckOutDate',
        'PublicSearchParameter_HotelList' => 'PublicSearchParameter.HotelList',
        'PublicSearchParameter_FilterRoomByPerson' => 'PublicSearchParameter.FilterRoomByPerson',
        'PublicSearchParameter_OnlyPPPrice' => 'PublicSearchParameter.OnlyPPPrice',
        'PublicSearchParameter_OnlyFGPrice' => 'PublicSearchParameter.OnlyFGPrice',
        'PublicSearchParameter_IsJustifyConfirm' => 'PublicSearchParameter.IsJustifyConfirm',
        'FacilityEntity_ChildrenAgeList' => 'FacilityEntity.ChildrenAgeList',
        'FacilityEntity_priceRange_lowPrice' => 'FacilityEntity.priceRange.lowPrice',
        'FacilityEntity_priceRange_highPrice' => 'FacilityEntity.priceRange.highPrice',
        'OrderBy_OrderName' => 'OrderBy.OrderName',
        'OrderBy_OrderType' => 'OrderBy.OrderType',
        'SearchCandidate_DateRange_Start' => 'SearchCandidate.DateRange.Start',
        'SearchCandidate_DateRange_End' => 'SearchCandidate.DateRange.End',
        'SearchCandidate_Occupancy_Adult' => 'SearchCandidate.Occupancy.Adult',
        'SearchCandidate_Occupancy_Child' => 'SearchCandidate.Occupancy.Child',
        'SearchCandidate_Occupancy_ChildInfo' => 'SearchCandidate.Occupancy.ChildInfo',
        'SearchCandidate_SearchTags' => 'SearchCandidate.SearchTags',
        'Settings_DisplayCurrency' => 'Settings.DisplayCurrency',
        'Settings_IsShowNonBookable' => 'Settings.IsShowNonBookable',
    ];


    public $Version = null;
    public $TransactionIdentifier = null;
    public $SearchCandidate_HotelID = null;
    public $SearchCandidate_RoomIDs = null;
    public $Settings_PrimaryLangID = null;
    public $Settings_ExtendedNodes = null;
    public $Settings_SearchTags = null;
    public $PagingSettings_PageSize = null;
    public $PagingSettings_LastRecordID = null;
    public $SearchCandidate_DateTimeRange_Start = null;
    public $SearchCandidate_DateTimeRange_End = null;
    public $SearchCandidate_StartTime = null;
    public $SearchCandidate_Duration = null;
    public $Settings_ShowDataRange = null;
    public $Settings_IsShowChangeDetails = null;
    public $Settings_DataCategory = null;
    public $SearchCandidate_SearchByType_SearchType = null;
    public $SearchCandidate_SearchByType_IsHaveHotel = null;
    public $SearchCandidate_SearchByCityID_CityID = null;
    public $SearchTypeEntity_SearchType = null;
    public $SearchTypeEntity_HotelCount = null;
    public $SearchTypeEntity_PageIndex = null;
    public $PublicSearchParameter_City = null;
    public $PublicSearchParameter_CheckInDate = null;
    public $PublicSearchParameter_CheckOutDate = null;
    public $PublicSearchParameter_HotelList = null;
    public $PublicSearchParameter_FilterRoomByPerson = null;
    public $PublicSearchParameter_OnlyPPPrice = null;
    public $PublicSearchParameter_OnlyFGPrice = null;
    public $PublicSearchParameter_IsJustifyConfirm = null;
    public $FacilityEntity_ChildrenAgeList = null;
    public $FacilityEntity_priceRange_lowPrice = null;
    public $FacilityEntity_priceRange_highPrice = null;
    public $OrderBy_OrderName = null;
    public $OrderBy_OrderType = null;
    public $SearchCandidate_DateRange_Start = null;
    public $SearchCandidate_DateRange_End = null;
    public $SearchCandidate_Occupancy_Adult = null;
    public $SearchCandidate_Occupancy_Child = null;
    public $SearchCandidate_Occupancy_ChildInfo = null;
    public $SearchCandidate_SearchTags = null;
    public $Settings_DisplayCurrency = null;
    public $Settings_IsShowNonBookable = null;

    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_HotelRef_HotelCode = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_StayDateRange_Start = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_StayDateRange_End = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RoomID = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RatePlanID = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_RatePlanCategory = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RatePlanCandidates_RatePlanCandidate_PrepaidIndicator = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RoomStayCandidates_RoomStayCandidate_GuestCounts_GuestCount_Count = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_RoomStayCandidates_RoomStayCandidate_Quantity = null;
    public $AvailRequestSegments_AvailRequestSegment_HotelSearchCriteria_Criterion_TPA_Extensions_LateArrivalTime = null;
    public $AvailRequestSegments_Version;
    public $AvailRequestSegments_TransactionIdentifier;
    public $HotelReservations = [];
    //public $HotelReservations_HotelReservation_ResGlobalInfo_DepositPayments = [];
    public $UniqueID = null;
    public $UniqueID_Type = null;
    public $UniqueID_ID = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_RoomTypes_RoomType_NumberOfUnits = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RoomID = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RatePlanCategory = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_PrepaidIndicator = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_RatePlans_RatePlan_RatePlanID = null;
    public $HotelReservations_HotelReservation_RoomStays_RoomStay_BasicPropertyInfo_HotelCode = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_PersonName = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_PersonName_Name = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_Telephone_PhoneTechType = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_Telephone_PhoneNumber = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson_ContactType = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_Profiles_ProfileInfo_Profile_Customer_ContactPerson = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_TPA_Extensions_LateArrivalTime = null;
    public $HotelReservations_HotelReservation_ResGuests_ResGuest_ArrivalTime = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_GuestCounts_GuestCount_Count = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_TimeSpan_Start = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_TimeSpan_End = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_Total_ExclusiveAmount = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_Total_InclusiveAmount = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_Total_CurrencyCode = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_ExclusiveAmount = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_InclusiveAmount = null;
    public $HotelReservations_HotelReservation_ResGlobalInfo_TPA_Extensions_TotalCost_CurrencyCode = null;
    public $ReservationPayment_ReservationID_Type = null;
    public $ReservationPayment_ReservationID_ID = null;
    public $ReservationPayment_PaymentDetail = null;
    public $ReservationPayment_Invoice = null;

    protected $dot;

    const HOTEL_STATIC_INFO_EXTENDED_NODES = [
        "HotelStaticInfo.GeoInfo",//重要
        "HotelStaticInfo.NearbyPOIs",//已废弃请下线处理
        "HotelStaticInfo.TransportationInfos",
        "HotelStaticInfo.Brand",
        "HotelStaticInfo.Group",
        "HotelStaticInfo.Ratings",//重要
        "HotelStaticInfo.Policies",//重要
        "HotelStaticInfo.AcceptedCreditCards",//重要
        "HotelStaticInfo.ImportantNotices",//重要
        "HotelStaticInfo.Facilities", //已废弃，请切换至FacilitiesV2节点
        "HotelStaticInfo.FacilitiesV2",//重要
        "HotelStaticInfo.Pictures",//重要
        "HotelStaticInfo.Descriptions",//重要
        "HotelStaticInfo.Themes",
        "HotelStaticInfo.ContactInfo",
        "HotelStaticInfo.HotelTags.IsBookable",//重要
        "HotelStaticInfo.HotelTags.IsHeCheng",
        "HotelStaticInfo.ArrivalTimeLimitInfo",//重要
        "HotelStaticInfo.DepartureTimeLimitInfo",//重要
        "HotelStaticInfo.ExternalFacility.Parking",
        "HotelStaticInfo.ExternalFacility.ChargingPoint",
        "HotelStaticInfo.BossInfos",
        "HotelStaticInfo.NormalizedPolicies.ChildAndExtraBedPolicy",//重要
        "HotelStaticInfo.HotelDiscount",
        "HotelStaticInfo.SellerShowInfos",
        "HotelStaticInfo.VideoItems",
        "HotelStaticInfo.NormalizedPolicies.MealsPolicy", //已废弃，请切换至MealsPolicyV2
        "HotelStaticInfo.NormalizedPolicies.MealsPolicyV2",//重要
        "HotelStaticInfo.HotelTags.ReservedData",
        "HotelStaticInfo.HotelPromotions",
        "HotelStaticInfo.HotelTaxRuleInfos",//重UniqueID要
        "HotelStaticInfo.SepecialServiceForChinese",
        "HotelStaticInfo.HotelFeatures",
        "HotelStaticInfo.HotelUsedNames",
        "HotelStaticInfo.BnBHotel", // 获取民宿酒店专有信息
        "HotelStaticInfo.ContactPersonInfos",
        "HotelStaticInfo.HotelCloseTime",  //酒店不营业时段
    ];

    const ROOM_TYPE_STATIC_EXTENDED_NODES = [
        //物理房型相关扩展节点
        "RoomTypeInfo.Facilities",
        "RoomTypeInfo.Pictures",
        "RoomTypeInfo.Descriptions",
        "RoomTypeInfo.ChildLimit",
        "RoomTypeInfo.BroadNet",
        "RoomTypeInfo.RoomBedInfos",
        "RoomTypeInfo.BnBHotel",    // 获取民宿房型专有信息
        "RoomTypeInfo.Smoking",//已废弃，请使用售卖房型吸烟信息
        "RoomTypeInfo.RoomTypeTags.ReservedData",
        //售卖房型相关扩展节点
        "RoomInfo.ApplicabilityInfo",
        "RoomInfo.Smoking",
        "RoomInfo.BroadNet",//已废弃，请使用物理房型层宽带信息，售卖房型层宽带信息暂停使用
        "RoomInfo.RoomBedInfos",
        "RoomInfo.RoomFGToPPInfo",
        "RoomInfo.ChannelLimit",
        "RoomInfo.ExpressCheckout",
        "RoomInfo.RoomTags",
        "RoomInfo.RoomGiftInfos",
        "RoomInfo.AreaApplicabilityInfo",
        "RoomInfo.RoomPromotions",
        "RoomInfo.HotelPromotions",
        "RoomInfo.MaskCampaignInfos",//Mask房型获取
        "RoomInfo.BookingRules",
        "RoomInfo.RoomTags.HotelDiscount",
        "RoomInfo.IsNeedCustomerTelephone",
    ];

    const ROOM_TYPE_STATIC_SEARCH_TAGS = [
        [
            "Code" => "IsOutputHiddenMaskRoom", //异地Mask房型获取
        ],
        [
            "Code"=> "IsOutputLimitDestinationRoom", //目的地Mask房型获取
        ]
    ];

    public function __construct()
    {
        $this->dot = new Dot;
    }

    protected function convertData()
    {
        foreach (self::RULES as $underlineRule => $dotRule) {
            if (null === $this->{$underlineRule}) {
                continue;
            }
            $this->dot->add($dotRule, $this->{$underlineRule});
        }
    }

    public function toArray()
    {
        $this->convertData();
        return $this->dot->all();
    }

    public function toJson()
    {
        $this->convertData();
        return $this->dot->toJson();
    }
}


class Test
{
    public function __construct($ctripClient)
	{
		$this->ctripClient = $ctripClient;
	}

    public function main()
    {
        $fetchOnePage = function (&$lastRecordId) {
            $request = new HotelRequest();
            $request->SearchCandidate_SearchByType_SearchType = self::SEARCH_TYPE;
            $request->SearchCandidate_SearchByType_IsHaveHotel = self::IS_HAVE_HOTEL;
            $request->PagingSettings_PageSize = self::PAGE_SIZE;
            $request->PagingSettings_LastRecordID = $lastRecordId;

            $response = $this->ctripClient->getAllCityInfo($request);

            if (!CtripClient::success($response) || empty($response['CityInfos']['CityInfo'])) {
                return Util::LOOP_API_EXCEPTION_STOP;
            }

            foreach ($response['CityInfos']['CityInfo'] as $city) {
				var_dump($city);
            }

            $lastRecordId = CtripClient::successAndReturnLastRecordId($response);
            if (empty($lastRecordId)) {
                unset($request, $response);
                return Util::LOOP_NORMAL_STOP;
            }

            unset($request, $response);
            return Util::LOOP_RUN_CONTINUE;
        };

        $lastRecordId = '';
        Util::loop($fetchOnePage, $lastRecordId);
    }
}

$test = new Test(new CtripClient());
$test->main();
```

* `CtripConfig`配置信息类
* `CtripToken`维护携程token类支持多进程并发获取token。
* `CtripClient`对外提供的客户端类
* `HotelRequest`对外提供的请求类
* `Util`工具类
* `Test`演示类

总结：支持分布式获取token；支持HotelRequest请求类参数配置；支持接口配置；支持接口限速配置；封装重试方法；封装分页循环获取方法。



