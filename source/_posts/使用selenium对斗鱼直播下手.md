---
title: 使用selenium对斗鱼直播下手
tags:
  - selenium
  - python
comments: true
date: 2021-06-23 14:18:35
categories: 爬虫
index_img: /2021/06/23/使用selenium对斗鱼直播下手/1.png
banner_img:
---

## 目标

获取斗鱼直播页面中，全部直播房间的房间信息。包括房间号，房间名称，房间标题，观看人数和直播预览图。

![](1.png)

## 实现代码

```python
# coding:utf-8
import time
import json
from selenium import webdriver


class Douyu(object):
    def __init__(self):
        # 实例化配置对象
        self.options = webdriver.ChromeOptions()
        self.options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36 Edg/91.0.864.54')
        self.url = 'https://www.douyu.com/directory/all'
        self.driver = webdriver.Chrome('D:/tools/chromedriver')

    def parse_data(self):
        room_list = self.driver.find_elements_by_xpath('//*[@id="listAll"]/section[2]/div[2]/ul/li/div')
        # print(len(room_list)) #120
        # 遍历房间列表，继续查找信息
        data_list = []
        for room in room_list:
            temp = {}
            temp['title'] = room.find_element_by_xpath('./a/div[2]/div[1]/h3').text
            temp['type'] = room.find_element_by_xpath('./a/div[2]/div[1]/span').text
            temp['owner'] = room.find_element_by_xpath('./a/div[2]/div[2]/h2/div').text
            temp['num'] = room.find_element_by_xpath('./a/div[2]/div[2]/span').text
            temp['img_src'] = room.find_element_by_xpath('./a/div[1]/div[1]/img').get_attribute("src")
            data_list.append(temp)
        return data_list

    def save_data(self, data_list):
        for data in data_list:
            print(data)

    def run(self):
        # url
        # driver
        # get
        self.driver.get(self.url)
        time.sleep(1)
        print(self.driver.find_element_by_xpath('//*[@id="listAll"]/section[2]/div[2]/div/ul/li[9]').get_attribute("aria-disabled"))
        while True:
            # parse
            data_list = self.parse_data()
            # save
            self.save_data(data_list)
            # next
            if self.driver.find_element_by_xpath('//*[@id="listAll"]/section[2]/div[2]/div/ul/li[9]').get_attribute("aria-disabled") == "false":
                # 模拟人为下拉
                self.driver.execute_script('scrollTo(0,10000)')
                self.driver.find_element_by_xpath('//*[@id="listAll"]/section[2]/div[2]/div/ul/li[9]').click()
                time.sleep(1)
            else:
                # close
                self.driver.quit()


if __name__ == '__main__':
    douyu = Douyu()
    douyu.run()
```

结果：

```python
{'title': '（长沙）唱歌跳舞主播求关注', 'type': '舞蹈', 'owner': '周小米Angel', 'num': '9.8万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCover/2021/03/10/7162561fddf51730b911ab668e13a7a3_big.png/webpdy1'}
{'title': '【PUBG SHOW】职业组', 'type': '绝地求生', 'owner': '绝地求生赛事3', 'num': '308.7万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/3939156_1606.png/webpdy1'}
{'title': '女野王连胜冲天梯第一~~', 'type': '王者荣耀', 'owner': '辜婉婉', 'num': '44.2万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/4766727_1611.png/webpdy1'}
{'title': '祝睡懵的小七5周年快乐！！！！！', 'type': '音乐', 'owner': '灵魂刀神', 'num': '101.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '直播间等你暴击×5', 'type': '颜值', 'owner': '睡懵的渣皇', 'num': '212万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/appCovers/2020/07/27/8922441_20200727190717_small.jpg/webpdy1'}
{'title': '别进来，小心薅你羊毛9636267', 'type': '颜值', 'owner': '羊羊baby呀', 'num': '10.2万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/appCovers/2021/06/05/9636267_20210605014050_small.jpg/webpdy1'}
{'title': '下午娱乐，晚上峡谷+画画', 'type': '英雄联盟', 'owner': '即将拥有人鱼线的PDD', 'num': '567.3万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCove_coverUpdate_2021-06-23_55b27af6c46d57fe008cc1a5fdc4264c.jpg/webpdy1'}
{'title': '看过星辰没看过你 本厅由佐岸哥哥冠名', 'type': '交友', 'owner': 'Zy一温香软玉', 'num': '17.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '6月23号下午3点开播', 'type': '户外', 'owner': '洪湖小肖', 'num': '190.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝皇妹四周年快乐，今晚OT', 'type': '户外', 'owner': '跳跳', 'num': '183万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '呆妹：有双倍亲密度加上限~袁袁子上线！', 'type': '和平精英', 'owner': '呆妹儿小霸王', 'num': '390.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'pigff【工资杯tyloo第一视角】', 'type': '绝地求生', 'owner': 'pigff', 'num': '340.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '预告丨26日18点 KPL春季赛总决赛', 'type': '王者荣耀', 'owner': '王者荣耀官方赛事', 'num': '310.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'Hi❤️欢迎听歌～', 'type': '音乐', 'owner': '张梦涵丶', 'num': '13.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '打开快乐大门 上午11点-5点', 'type': '舞蹈', 'owner': '野鸡娘娘', 'num': '12.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '淑怡：来了嗷', 'type': '英雄联盟', 'owner': '不2不叫周淑怡', 'num': '291.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '初晨 虔诚 762 KOKO 拖米', 'type': '王者荣耀', 'owner': 'DL丶拖米', 'num': '267.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '白天坐牢军训，晚上盗贼之海!', 'type': 'DOTA2', 'owner': 'yyfyyf', 'num': '257万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '别翻了，就这坐下吧', 'type': '颜值', 'owner': '星星雪糕', 'num': '11.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '相遇能不能重写9681437', 'type': '颜值', 'owner': '思唯因', 'num': '2.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '美好的一天从连麦小姐姐开始！', 'type': '英雄联盟', 'owner': '南波儿大魔王丶', 'num': '245.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '长沙密室冲冲冲！！！', 'type': '英雄联盟', 'owner': '智勋勋勋勋', 'num': '236.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '17点LPL夏季赛JDGvsV5', 'type': '英雄联盟', 'owner': '英雄联盟赛事', 'num': '236.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '杭州 00 天秤 好骗', 'type': '颜值', 'owner': '鬼鬼不是鬼aoa', 'num': '12.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【大明攒机】电脑配件 电源选择', 'type': '数码科技', 'owner': '上海大明海', 'num': '36.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '十指操作，初露锋芒', 'type': '和平精英', 'owner': '艺帝帝', 'num': '235.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'Gemini: 6.6 北诗 小夜 不弃', 'type': '王者荣耀', 'owner': 'MrGemini', 'num': '232.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '孙悟空：峡谷有善口技者', 'type': '英雄联盟', 'owner': '孙悟空丨兰林汉', 'num': '232万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝皇妹周年快乐', 'type': '颜值', 'owner': 'TD丶正直博', 'num': '147.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '夜晚拥有星星 云朵拥有雨滴', 'type': '颜值', 'owner': '小花不敢吃鱼了', 'num': '130.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '下午三点准时开播~', 'type': '英雄联盟', 'owner': '芜湖大司马丶', 'num': '231.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '这才叫麻将，暴躁杠开集锦！', 'type': '欢乐麻将', 'owner': '靓旭', 'num': '224.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季5排冲冲冲！！！', 'type': '王者荣耀', 'owner': 'DYG丶久诚', 'num': '218.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '业余唱歌爱好者', 'type': '二次元', 'owner': '趣趣er', 'num': '123.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '距离0.1km/感谢“改名，废话”宠冠！', 'type': '交友', 'owner': 'Yw一如约而至', 'num': '97.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新英雄云樱百分百胜率上国服！', 'type': '王者荣耀', 'owner': 'A老回', 'num': '217.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【直播】CFML Q9 0:1 ES', 'type': 'CF手游', 'owner': 'CF手游官方直播间', 'num': '203.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '快乐云缨上线 冲国服教学', 'type': '王者荣耀', 'owner': '白白白白白3', 'num': '203.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '我回来啦！！ 681888', 'type': '颜值', 'owner': 'Skr丶大伟伟', 'num': '97.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '如鹿归林 如舟靠岸 7009686', 'type': '颜值', 'owner': '紫薇同学', 'num': '94.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '涵涵：CFHD斗鱼赏金活动火热进行！', 'type': 'CFHD', 'owner': '画饼李', 'num': '202.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'HELLO我是张老板阿~冲鸭！', 'type': '和平精英', 'owner': '张逗逗就是张老板阿', 'num': '194.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【猛男】单人4排最激情单口相声细节教学', 'type': '绝地求生', 'owner': '邦Sa', 'num': '190.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝巨蟹哥朝圣哥生日快乐', 'type': '颜值', 'owner': '歌神韦爵爷', 'num': '90.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '小胖子宝儿', 'type': '颜值', 'owner': '鲭宝儿', 'num': '90.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '19点 CFPL Q9 vs ES', 'type': '穿越火线', 'owner': '穿越火线运营团队', 'num': '189.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '余小C：峡谷之巅冲前500！', 'type': '英雄联盟', 'owner': '余小C真的很强', 'num': '187.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'JJ斗地主 488743', 'type': 'JJ斗地主', 'owner': 'JJ斗地主官方', 'num': '185.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '颜值区倒数第一。', 'type': '颜值', 'owner': '小悦同学ovo', 'num': '86万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '去成都约小姐姐吃饭', 'type': '户外', 'owner': '逐梦兄弟', 'num': '79.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【头号玩家】最后一把上战神', 'type': '和平精英', 'owner': '追梦战神泼痞', 'num': '184.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '2021CRL六月月度决赛 第2天', 'type': '皇室战争', 'owner': '皇室战争官方直播间', 'num': '183.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '2021BSC六月月度决赛 中国大陆赛区', 'type': '荒野乱斗', 'owner': '荒野乱斗官方直播间', 'num': '182.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【持续更新中】饭真香', 'type': '美食', 'owner': '徐大sao', 'num': '77.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '9815091万合二台，《鲜肉老师》上线', 'type': '原创IP', 'owner': '万合出品', 'num': '76.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '放逐三妹~欢乐下饭', 'type': '英雄联盟', 'owner': '放逐大帝灬', 'num': '182.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '先来上一个王者的', 'type': '王者荣耀', 'owner': '少年不太冷2', 'num': '177.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '天不生我王纪超，炮道万古如长夜', 'type': '英雄联盟', 'owner': '王纪超666', 'num': '172.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝我秀哥生日快乐～', 'type': '颜值', 'owner': '果冻cc007', 'num': '71.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝巨蟹哥生日快乐~~~~', 'type': '颜值', 'owner': '梅夕不是梅西啊', 'num': '70.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【橙子】最钢竞技单四全程高能视角盛宴', 'type': '绝地求生', 'owner': '神秘嘉宾橙子', 'num': '166.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '少囧の游戏教室', 'type': '绝地求生', 'owner': '少囧丶', 'num': '166.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【骚某某】场均50杀，世界第一', 'type': '绝地求生', 'owner': '骚某某', 'num': '166.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【特辑】追龙已成往事，王总终成大局！', 'type': '原创IP', 'owner': '特辑电影院', 'num': '69.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '本房间由“ bs 小夏 问路”冠', 'type': '交友', 'owner': '小象一我喜欢你', 'num': '69.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【挑战你的不开心】', 'type': '绝地求生', 'owner': 'imxiaoxin', 'num': '165.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '每天躺鸡好难受啊', 'type': '热门手游', 'owner': '蓝天白云的卡卡', 'num': '165万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【菊花】无挂服顶级运营天花板', 'type': '绝地求生', 'owner': '顶级菊宝a', 'num': '164.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【正在解说光头琦玉】饭点陪聊', 'type': '原创IP', 'owner': '小陶气说动漫', 'num': '68.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【交友】招收女主持一交友一厅', 'type': '交友', 'owner': '爱约一只如初见', 'num': '68.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '无敌战神在线被老6', 'type': '绝地求生', 'owner': '轻语丶619', 'num': '164.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '直播直播直播 5688727', 'type': '王者荣耀', 'owner': 'eStarPro花海', 'num': '164.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '单排冲击王者！', 'type': '王者荣耀', 'owner': '不可一世杀手', 'num': '163.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '本厅由“东月老板”冠名', 'type': '交友', 'owner': '小象一念你如初', 'num': '66.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【最新视频】大口吃肉，太过瘾了', 'type': '美食', 'owner': '苗大冬', 'num': '64.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '冲第一个上国服榜云缨！', 'type': '王者荣耀', 'owner': '陈比克rr', 'num': '163.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '五国服顶级发育路，冲', 'type': '王者荣耀', 'owner': 'BOA小强', 'num': '163.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【直播】CFHD战神杯小组赛B组DAY3', 'type': 'CFHD', 'owner': 'CFHD运营团队', 'num': '162.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '白客、孔连顺主演《鲜肉老师》登陆斗鱼！', 'type': '趣生活', 'owner': '万合鲜肉老师', 'num': '61.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【牛叔】N分钟带你看完大片', 'type': '原创IP', 'owner': '牛叔说电影', 'num': '61.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【2021PCS4】18点东亚赛区', 'type': '绝地求生', 'owner': '斗鱼绝地赛事', 'num': '160.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '云缨乱送局', 'type': '王者荣耀', 'owner': 'DYG丶小义', 'num': '158.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '巅峰赛冲冲冲！！！~', 'type': '王者荣耀', 'owner': '陈正正正Cat', 'num': '156.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【谭sir】火遍全网的“二仙桥”', 'type': '趣生活', 'owner': '谭乔V', 'num': '60.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【正版】经典小品，下饭必备', 'type': '趣生活', 'owner': '辽台好物种草', 'num': '59.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '叶知秋：下午好', 'type': '绝地求生', 'owner': '叶知秋Vanessa', 'num': '156万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季五排冲王者啦..', 'type': '王者荣耀', 'owner': '骚易丶', 'num': '155.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【直播】TI10东欧区预选赛', 'type': 'DOTA2', 'owner': '完美世界电竞频道', 'num': '153.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '快乐的一天从这里开始 7327546', 'type': '颜值', 'owner': '杨薰同学', 'num': '59.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '超小厨的直播间', 'type': '美食', 'owner': '超小厨', 'num': '59.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新人主播第一天直播第一天玩游戏', 'type': 'CFHD', 'owner': '性空z', 'num': '148.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季冲冲冲！！ 1981932', 'type': '王者荣耀', 'owner': '音七Simple', 'num': '146.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '《路人王带妹》', 'type': '和平精英', 'owner': 'A扬丷', 'num': '144.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '你来我相信你不会走', 'type': '舞蹈', 'owner': '一渔1', 'num': '59.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '迷人可爱又壮实', 'type': '颜值', 'owner': '大艺术嘉carrie', 'num': '58.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服马超国服关羽国服露娜专场', 'type': '王者荣耀', 'owner': 'DYG易瞳', 'num': '142.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '今天也可以反着压~', 'type': '穿越火线', 'owner': '白鲨石头丶', 'num': '140.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '上王者冲巅峰赛', 'type': '王者荣耀', 'owner': '国宝佳悦', 'num': '139.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '心有猛虎 嗷呜嗷呜~！', 'type': '颜值', 'owner': '小卷呐', 'num': '57.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '繁星哥霸气冠名/招全职主持！！', 'type': '交友', 'owner': '小象一勿忘初心', 'num': '55.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '❤摸摸喜酱头万事不用愁', 'type': '王者荣耀', 'owner': '喜衣机', 'num': '138.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '我可能玩了假的马里奥', 'type': '马里奥制造', 'owner': '超级小桀', 'num': '137.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '速战速决！昨天1神话4史诗=7倍', 'type': 'DNF', 'owner': 'DNF黑一阿旭', 'num': '136.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '下午见呀下午见', 'type': '颜值', 'owner': '早安酱ii', 'num': '55.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '直-3倍上限＋双倍亲密度 ~', 'type': '电台新声', 'owner': '孙紫涵丶', 'num': '55.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新一：大师以上，我必一飞冲天直接王者！！', 'type': '英雄联盟', 'owner': 'Xinyi新一丶', 'num': '135.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '❤呆萌懒妹嘎嘎乱杀❤', 'type': '王者荣耀', 'owner': '苏小懒38D', 'num': '132.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新英雄云缨顶级理解全面剖析', 'type': '王者荣耀', 'owner': '有痕ysys', 'num': '131.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【战神杰】满是经典', 'type': '一起看', 'owner': '神秘水友丶战神杰', 'num': '53.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '田园仙境有佳人，李子柒最新视频已更新', 'type': '美食', 'owner': '李子柒liziqi', 'num': '53.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '石原：月底冲分！！！！', 'type': '火影忍者', 'owner': '爱吃鱼的石原', 'num': '128.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '重温红色经典 汲取前行力量', 'type': '正能量', 'owner': '政达光明', 'num': '128.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【QQQ】前职业选手Q7，秀！', 'type': '绝地求生', 'owner': '气人主播QQQ', 'num': '125.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '如鲸向海：由靓仔 萝莉 灵犀冠名！', 'type': '交友', 'owner': '小象一如鲸向海', 'num': '53.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【每日更新】感谢大家，一起好好吃饭', 'type': '美食', 'owner': '兴森一家', 'num': '53.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【认证号王牌1V4】新版本大揭幕！', 'type': '和平精英', 'owner': 'Dy合群', 'num': '124.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '滴滴滴！牛根儿发车啦！', 'type': 'DNF', 'owner': '一阵雨不是一阵奶', 'num': '123.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【新版本体验】', 'type': '和平精英', 'owner': '脆油条丷', 'num': '122.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '已更新，不定期吐槽美国', 'type': '趣生活', 'owner': '我是郭杰瑞V', 'num': '50.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '六月好一点', 'type': '颜值', 'owner': '安小安安呀', 'num': '50.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【新版本体验】', 'type': '和平精英', 'owner': '脆油条丷', 'num': '122.1万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/1322998_1606.png/webpdy1'}
{'title': '预告丨23日17点KPL总决赛练兵', 'type': '王者荣耀', 'owner': '斗鱼王者荣耀微博杯', 'num': '122.1万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCove_coverUpdate_2021-06-23_c3c0f8f1612e84c87acb4c81584583ea.jpg/webpdy1'}
{'title': '国服赵云打野90胜率', 'type': '王者荣耀', 'owner': '声优赵猥琐', 'num': '121.6万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/2205764_1611.png/webpdy1'}
{'title': '双倍经验~ 记得打卡哟~', 'type': '颜值', 'owner': '伊伊不可爱', 'num': '49.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【开心一刻】宋小宝经典小品轮播', 'type': '趣生活', 'owner': '宋小宝小品', 'num': '49.5万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/9817729_1612.png/webpdy1'}
{'title': '马牛牛：十天上大师DAY2', 'type': '绝地求生', 'owner': '47号Gamer', 'num': '120.8万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/2461752_1606.png/webpdy1'}
{'title': '新赛季三排冲王者', 'type': '王者荣耀', 'owner': '特特zzz', 'num': '120.1万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/8813116_1612.png/webpdy1'}
{'title': '少点看弹幕认真冲第一', 'type': '英雄联盟', 'owner': 'impOUO', 'num': '119.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【开心一刻】赵本山经典小品轮播', 'type': '趣生活', 'owner': '赵本山经典小品', 'num': '49.5万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/9817716_1612.png/webpdy1'}
{'title': '不过四星不下播！！7189349', 'type': '户外', 'owner': '户外和平大使', 'num': '49.1万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/7189349_1611.png/webpdy1'}
{'title': '最强画质COD+派派+塔科夫！！！！', 'type': '绝地求生', 'owner': '斯祥丶', 'num': '119万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/2311698_1606.png/webpdy1'}
{'title': '单排不上王者不下播，输一把发红包！', 'type': '王者荣耀', 'owner': '伽罗王丶', 'num': '118.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服第一妲己：轻松上分', 'type': '王者荣耀', 'owner': '国服老狐狸', 'num': '116.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【最新已更】嗨吃小强，陪你下饭！', 'type': '美食', 'owner': '吃饭啦光小强', 'num': '48.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【正版】经典小品轮播电视台', 'type': '趣生活', 'owner': '辽视欢乐集结号', 'num': '46.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '粉丝牌跟车上战神,晚七点送CDK', 'type': 'COD手游', 'owner': '叫老公别叫老郭', 'num': '116.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '微笑：峡谷冲大师！！', 'type': '英雄联盟', 'owner': '微笑', 'num': '116.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服第一生化 有问必答 视觉体验', 'type': 'CFHD', 'owner': '可杰大魔王', 'num': '114.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '打卡日/今天我也爱你', 'type': '交友', 'owner': '考拉一人间酒馆', 'num': '46.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '赶集收土鸭蛋', 'type': '户外', 'owner': '叶子户外', 'num': '44.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '燎原神枪第十八代传人 上王者', 'type': '王者荣耀', 'owner': '小乐碎片', 'num': '112.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '请问嘻嘻哈哈能上万一么？', 'type': '炉石传说', 'owner': '衣锦夜行', 'num': '112.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '训练赛一起快乐练枪', 'type': '和平精英', 'owner': 'Tianba丶顾居居', 'num': '111.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '真人真比封面好看', 'type': '颜值', 'owner': '凌晨儿Mini', 'num': '44万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '录-孤立无援就自成宇宙吧~', 'type': '电台新声', 'owner': '阿睿ARrr', 'num': '42.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '奶粉：班德尔城大师教学屠杀局！', 'type': '英雄联盟', 'owner': '主播油条', 'num': '111.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '双服第七峡谷冲第一', 'type': '英雄联盟', 'owner': '天灰灰qaq', 'num': '110.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '彩排24号云顶2周年晚会，兄弟们别进！！', 'type': 'lol云顶之弈', 'owner': '卷子丶', 'num': '110.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '淑洁:今天回武汉了', 'type': '户外', 'owner': '百变姑姑肖淑洁', 'num': '41.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【江苏】月底冲业绩', 'type': '颜值', 'owner': '韩小沫呀', 'num': '41万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '继续阿凡达！~~', 'type': '永劫无间', 'owner': '女流66', 'num': '110.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '云缨胎教系列冲国服 我看谁不学', 'type': '王者荣耀', 'owner': '林豆豆y', 'num': '109.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '二台:思路清晰教你稳定吃鸡！', 'type': '英雄联盟', 'owner': '仙凡哥哥丶', 'num': '107.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【小片片】片荒的日子就找小片片！', 'type': '原创IP', 'owner': '小片片说大片', 'num': '40.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '2点左右播～下午见', 'type': '舞蹈', 'owner': '小梦露mm', 'num': '40.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '妖路整活，噜子上线！', 'type': 'DOTA2', 'owner': 'Amore婉', 'num': '106.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【赛事回看】Gambit专场', 'type': 'CS:GO', 'owner': '斗鱼CSGO赛事主频道', 'num': '104.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '十点半,不见不散!!!', 'type': 'DOTA2', 'owner': '谢彬DD', 'num': '104.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '斗鱼大事件', 'type': '交友', 'owner': '奇点一可口可乐', 'num': '39.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【最新视频已更新】想看料理，就来看大姨！', 'type': '美食', 'owner': '绵羊料理', 'num': '39.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '石页:请大家不要窥屏BAN英雄！谢谢！', 'type': '英雄联盟', 'owner': '石页的第一根矛s', 'num': '103.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '黄金圣光战神不死流剑魔安排！', 'type': 'lol云顶之弈', 'owner': '阿涛皎月Carry', 'num': '103.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '巅峰永无止境！！', 'type': '王者荣耀', 'owner': '深圳DYG易峥', 'num': '103.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '听说老婆女友最爱农国栋做的菜', 'type': '美食', 'owner': '主厨农国栋', 'num': '39.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '距您0.03km/“服务”独家冠名', 'type': '交友', 'owner': 'XK一万有瘾力', 'num': '39.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '皮鸭助手已上线', 'type': '和平精英', 'owner': '老皮鸭OVO', 'num': '103.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季五排', 'type': '王者荣耀', 'owner': 'eStarPro无铭', 'num': '103.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '学习嘴甜的第一天 双倍亲密度~', 'type': '英雄联盟', 'owner': '小苏菲', 'num': '102.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '努力渡劫冲鸭 8733064', 'type': '音乐', 'owner': '阿允唱情歌', 'num': '38.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '好久不见呀 6291080', 'type': '颜值', 'owner': '瑄璐', 'num': '38.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '好好上分光速上王者', 'type': '英雄联盟', 'owner': '格局OoO', 'num': '102.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【令北】办卡加时长挑战 第22天', 'type': '绝地求生', 'owner': '正经人令北', 'num': '102.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '另类玩法一双喷吃鸡', 'type': '和平精英', 'owner': '苟分怪', 'num': '102万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '月光会把你带到我身边~', 'type': '颜值', 'owner': '胖冰不可爱', 'num': '37.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【招主持】厅由小8 新兰哥 鞋带哥冠名', 'type': '交友', 'owner': '小象一浮生未歇', 'num': '37.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '小亮：最强单人四排！', 'type': '绝地求生', 'owner': '左小亮丶', 'num': '101.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【直播】TI10东欧预选赛副舞台', 'type': 'DOTA2', 'owner': '完美世界电竞频道3', 'num': '100.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新英雄云缨冲国服！', 'type': '王者荣耀', 'owner': '羽辰鸽鸽丶', 'num': '100.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '会修一年吗 一年能等吗', 'type': '户外', 'owner': '你的网友小董', 'num': '37.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '“精神小伙”蛋黄派最新自制硬菜', 'type': '美食', 'owner': '记录生活的蛋黄派1', 'num': '37.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '二台馒头：端游转手游的第13天', 'type': 'COD手游', 'owner': '皮皮鲨FPS', 'num': '99.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【有车位】萌妹开心上大分！', 'type': '王者荣耀', 'owner': '欣小菜', 'num': '95.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '职业选手玩云樱', 'type': '王者荣耀', 'owner': 'eStarPro橘子', 'num': '94.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '祝皇妹周年快乐~~！', 'type': '颜值', 'owner': '招财猫呢', 'num': '37.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '水库网鱼，明天赶集！', 'type': '户外', 'owner': '长沙三湘独立团', 'num': '37.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '赛季初5排上个王者', 'type': '王者荣耀', 'owner': '小小xxl', 'num': '94.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '峡谷7天必王者', 'type': '英雄联盟', 'owner': 'KZH盲僧', 'num': '94.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【72战神火力车】猛摇！', 'type': '和平精英', 'owner': '追梦战神清风', 'num': '94.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【敬汉卿】将作死进行到底', 'type': '趣生活', 'owner': '敬汉卿250', 'num': '37万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '美食作家王刚V的直播间', 'type': '美食', 'owner': '美食作家王刚V', 'num': '36.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '科技美学的直播间', 'type': '数码科技', 'owner': '科技美学中国', 'num': '93.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '别乐我别乐我', 'type': '绝地求生', 'owner': '17小北', 'num': '89.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'jjp:桀目效果', 'type': 'APEX英雄', 'owner': 'Jiepai桀派', 'num': '89万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '你会遇到很多星星但只心动一个月亮', 'type': '颜值', 'owner': '蔡娜妹妹', 'num': '36.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '言念君子 温其如玉', 'type': '颜值', 'owner': '翼遥遥丶', 'num': '36.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '四排战术全能王', 'type': '绝地求生', 'owner': '天赐解说丷', 'num': '88.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 's1首位百星 格斗大赛冠军。', 'type': '航海王热血航线', 'owner': '酒九冬', 'num': '87.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '郑州80it：618电脑活动开启', 'type': '数码科技', 'owner': '80it电脑网', 'num': '87万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '万般皆苦 唯有自渡', 'type': '颜值', 'owner': '珂珂宝丶', 'num': '36.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '哥哥对妹妹这样姐姐会生气吗', 'type': '颜值', 'owner': '玉玉子想吃饱', 'num': '35.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【湖北ju王】手机天花板系列', 'type': '和平精英', 'owner': '一玥阿', 'num': '86.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '中路英雄海王 对线教学show', 'type': 'LOL手游', 'owner': '西瓜太浪xx', 'num': '86万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季上王者冲', 'type': '王者荣耀', 'owner': '深圳DYG星宇', 'num': '85.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '渝万七厅-金牌主持：谭洁', 'type': '开黑', 'owner': '渝万丶派单七厅', 'num': '35.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '八卦兽:dy仅存职业八卦~', 'type': '户外', 'owner': '薛叫兽', 'num': '35万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【2021PGL夏季赛训杯】', 'type': '绝地求生', 'owner': '绝地求生赛事4', 'num': '85万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '醒目：勇敢牛牛 不怕困难', 'type': '绝地求生', 'owner': 'Cpt小醒目', 'num': '84.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'YJJ 永劫无间', 'type': '绝地求生', 'owner': 'yjjimpaopao', 'num': '84.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【姽婳】九亿少男的梦', 'type': '一起看', 'owner': '一条小姽婳', 'num': '35万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '俄罗斯少女', 'type': '户外', 'owner': '俄罗斯教父', 'num': '34.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '巅峰赛上大分 9177171', 'type': '王者荣耀', 'owner': 'DYG萧玦', 'num': '84.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '空调直播间\u3000凉快！', 'type': '了不起的修仙模拟器', 'owner': '主播杨树', 'num': '84.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '第一天五排主播队冲王者', 'type': '王者荣耀', 'owner': '音子tut', 'num': '83.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【交友】斗鱼颜值交友一厅，气氛一厅', 'type': '交友', 'owner': '爱约一爱约之夏', 'num': '34.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【柯南】专业解说柯南直播间', 'type': '原创IP', 'owner': '我是一只西瓜啊', 'num': '34.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '无尽诺手版本1V5答案', 'type': '英雄联盟', 'owner': '旋转的老刘诺手', 'num': '83.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '蛙神：今天不上50星不下播', 'type': '王者荣耀', 'owner': '小小青蛙笑i', 'num': '82.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '巅峰2526.守榜前十', 'type': '王者荣耀', 'owner': '区区一只箐笙', 'num': '81.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '经典小品相声乐不停', 'type': '趣生活', 'owner': '吉林卫视v', 'num': '34.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '等一个奥特曼！！', 'type': '舞蹈', 'owner': 'Lulala拉拉', 'num': '33.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季上王者 9177702', 'type': '王者荣耀', 'owner': 'DYG丶Giao', 'num': '81.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季玩具无牌车', 'type': '王者荣耀', 'owner': 'eStarPro无心', 'num': '81.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '渝乐耍大牌，每晚18:00与大屏同步直播', 'type': 'JJ斗地主', 'owner': '渝乐耍大牌', 'num': '81.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '云栖丶开黑一厅', 'type': '开黑', 'owner': '云栖丶开黑一厅代号哥', 'num': '33.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '稳定努力主持+', 'type': '交友', 'owner': 'XK一予你欢喜', 'num': '33.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '小13：YYDS~~~~~', 'type': '欢乐麻将', 'owner': '头号玩家丶B', 'num': '81.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '周三快乐！！！！', 'type': '自走棋', 'owner': '小红豆阿', 'num': '81.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '沙皇：请大家监督我训练hhh', 'type': 'COD手游', 'owner': '沙皇StaFenGraD', 'num': '80.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '已更新，德国人在中国', 'type': '趣生活', 'owner': '阿福ThomasV', 'num': '33.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '理性男友：全麦单身~', 'type': '交友', 'owner': 'Hi一理性男友', 'num': '33.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '主播打野，有车位呐 ♥~', 'type': '王者荣耀', 'owner': '小龙女OuO', 'num': '80.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '骆歆：你好吗？', 'type': '英雄联盟', 'owner': '骆歆冲冲冲', 'num': '80.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '湫兮：五排冲王者', 'type': '王者荣耀', 'owner': '湫兮ovo', 'num': '80.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '或许你喜欢胖嘟嘟（重庆）', 'type': '舞蹈', 'owner': '还是来福儿', 'num': '33万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【交友】招收优质交友女主持', 'type': '交友', 'owner': '爱约一缘来是你', 'num': '32.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '朗基努斯大头！1张就进决赛', 'type': '英雄联盟', 'owner': '中年人拳师七号', 'num': '80.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '⭐A测全服第七 箐英比赛中', 'type': 'LOL手游', 'owner': '阿超cccccccccc', 'num': '79.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '异灵术：今日主旋律 上分', 'type': '炉石传说', 'owner': '异灵术老师', 'num': '79.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '主播距离你0.3km…', 'type': '颜值', 'owner': '一林绝美的', 'num': '32.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '郑州 御姐 一笑你就化了', 'type': '舞蹈', 'owner': '我叫李菲儿', 'num': '32.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季刘备连胜五排上王者！', 'type': '王者荣耀', 'owner': '刘皇叔是个弟弟', 'num': '78.8万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCove_coverUpdate_2021-06-23_2473c2cd370f5d55596004ffda261ca6.png/webpdy1'}
{'title': '新赛季带粉开冲!', 'type': '王者荣耀', 'owner': '苏小沫不胖', 'num': '77.2万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCove_coverUpdate_2021-06-23_36ed287169b28abd69f9d7caf9ddef11.jpg/webpdy1'}
{'title': '表弟：第一帅盲僧刀妹', 'type': '英雄联盟', 'owner': '疯狗炼金', 'num': '76万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/338281_1606.png/webpdy1'}
{'title': '周一拉填榜拉~', 'type': '颜值', 'owner': '夏宝宝宝宝宝丶', 'num': '32.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '纯天然无公害主播9390137', 'type': '颜值', 'owner': '黎可妮', 'num': '32.2万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCover/2021/05/08/9ebe5b42a56accdcff74ad383f2004aa_big.png/webpdy1'}
{'title': '12点前上2800分上不去送2800！', 'type': '永劫无间', 'owner': '狐狸不太Sao', 'num': '75.9万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/699689_1606.png/webpdy1'}
{'title': '国一司马懿，赛季第一天冲啊', 'type': '王者荣耀', 'owner': '环斐丶蓝羽', 'num': '75.4万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/4749583_1606.png/webpdy1'}
{'title': '今天玩下底特律变人~~~', 'type': 'DOTA2', 'owner': '820邹倚天', 'num': '75.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '交友最美颜值厅 招收冠名大哥 全职主持', 'type': '交友', 'owner': '小象一神仙姐姐i', 'num': '32.1万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/appCovers/2021/05/22/9799138_20210522133515.jpg/webpdy1'}
{'title': '凉凉主播也需要疼爱QAQ', 'type': '颜值', 'owner': '未三岁i', 'num': '32.1万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/appCovers/2021/03/27/4400186_20210327185315_small.jpg/webpdy1'}
{'title': 'zard1991', 'type': 'DOTA2', 'owner': 'zard1991', 'num': '74万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/60937_1606.png/webpdy1'}
{'title': '新赛季的上分秘诀', 'type': '王者荣耀', 'owner': 'RW侠Djie', 'num': '73.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【月光】专业看号规划/深渊', 'type': '原神', 'owner': '天书月光', 'num': '72.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '每一天都是新的际遇', 'type': '颜值', 'owner': '绒绒儿ovo', 'num': '31.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【国语点唱】世界这么大感谢遇见你❤', 'type': '二次元', 'owner': '睡不醒的妮儿', 'num': '31.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '庆祝建党100周年 686877', 'type': '正能量', 'owner': '鱼公益善', 'num': '72.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国一akbp短剑8指相声山在线讲解', 'type': 'COD手游', 'owner': '山山鸭', 'num': '72.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '全民PK赛梦幻西游手游，221869~', 'type': '梦幻手游', 'owner': '小儿郎丶丶', 'num': '71.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'YJ-浪漫之爱‘四四神/靓仔仔’冠', 'type': '交友', 'owner': '英爵一浪漫之约', 'num': '31.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '蕉蕉不焦虑~~~', 'type': '舞蹈', 'owner': '撒个蕉', 'num': '31.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '深夜伴侣-五指狙神', 'type': '和平精英', 'owner': '炎师傅', 'num': '71.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '20冰雪稳定开放', 'type': '传奇', 'owner': '20冰雪dy6512', 'num': '70.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季暴力女野王带粉冲呀', 'type': '王者荣耀', 'owner': '乔公举', 'num': '70万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '本房已开启一元粉丝牌！！！', 'type': '交友', 'owner': 'YAQ一夜色星河', 'num': '31.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【交友】凯美阳：希望美阳天天开心', 'type': '交友', 'owner': '爱约一不期而遇', 'num': '31.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '百段野王巅峰赛教学', 'type': '王者荣耀', 'owner': '潇潇暴龙兽', 'num': '69.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '帮小咪助威投票领DNF道具咯', 'type': 'DNF', 'owner': '大红神o猫小咪', 'num': '69.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【海岛争爸赛】三十九岁天才少年', 'type': '和平精英', 'owner': '浪浪直男吖', 'num': '69.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【最新更新】武侠浪子的美食分享，过瘾', 'type': '美食', 'owner': '山药视频', 'num': '30.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'I NEED YOU~新人主播第三天~', 'type': '颜值', 'owner': '小七就是小七宝', 'num': '30.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【年鹏】今天有事，跟教练们请天假，明天播', 'type': '穿越火线', 'owner': '汉宫年鹏Enpi', 'num': '69万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '愿山河无恙，人间皆安！加油，广东！', 'type': '数码科技', 'owner': 'OC频道包大人', 'num': '69万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '晚上好啊 我来了 37807', 'type': '格斗游戏', 'owner': '战神_河池VR', 'num': '69万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '朱一旦顶级团队C座802，职场大爆炸！', 'type': '趣生活', 'owner': 'C座802', 'num': '30.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '诶嘿9822129', 'type': '二次元', 'owner': '安晓呀', 'num': '30.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '战斗战斗战斗', 'type': '英雄联盟', 'owner': '滔搏369zz', 'num': '68.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '《火线犀利说》：观看抽永久冠军之怒', 'type': '穿越火线', 'owner': '穿越火线赛事', 'num': '67.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季和七七一起双排上王者', 'type': '王者荣耀', 'owner': '深海ToT', 'num': '67.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '迈克尔和哥哥林肯的越狱计划', 'type': '原创IP', 'owner': '3说电影', 'num': '29.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '旅行老司机 爱上户外', 'type': '户外', 'owner': '老狼雄起a', 'num': '29.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【18ym】保持良好的心态', 'type': 'CS:GO', 'owner': '18goYm', 'num': '67.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '九寨沟看房团上线', 'type': '剑网3', 'owner': '剑网3官方直播', 'num': '67.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '峡谷超级艾琳 单排冲王者 嗷嗷乱杀', 'type': '王者荣耀', 'owner': '狸子3岁', 'num': '66.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【最新更新】小哥，陪你下饭', 'type': '美食', 'owner': 'YESHI小哥', 'num': '29.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '本厅由最最最帅气的店长冠名~', 'type': '交友', 'owner': '小象一甜心蜜语', 'num': '29.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '[冰子]来来龙盒碰瓷免单了，有深渊打造！', 'type': 'DNF', 'owner': 'DNF枪魂冰子', 'num': '65.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '冯雨大王：欢乐时光已经开始很久了！', 'type': 'LOL手游', 'owner': '驴酱的冯雨大王', 'num': '65.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '魔法部长小罗：峡谷第一好抓', 'type': '英雄联盟', 'owner': '魔法部长小罗', 'num': '65万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '「橙幕」「最人气港剧直播间」', 'type': '一起看', 'owner': '橙幕Orange丶', 'num': '29.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '正宗农村风味~馋~~~', 'type': '美食', 'owner': '食味阿远', 'num': '29.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季第一天上王者 8745735', 'type': '王者荣耀', 'owner': '沐沐Ccccc', 'num': '65万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '法系边路九少', 'type': '王者荣耀', 'owner': 'BOA全能哔', 'num': '64.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '星耀三开打今天就能国1', 'type': '王者模拟战', 'owner': 'KSSC抓只好棋', 'num': '64.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '本周北京出差啦 不定时播', 'type': '音乐', 'owner': '甜小沐', 'num': '29.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '如火如荼由偏爱哥，强哥冠名（招募主持）', 'type': '交友', 'owner': '小象一如火如荼', 'num': '29.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'eStarPro今屿的直播间', 'type': '王者荣耀', 'owner': 'eStarPro今屿', 'num': '64万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '拖米 初晨 虔诚 762 新赛季5排', 'type': '王者荣耀', 'owner': 'TESKoko', 'num': '62.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【备战PEL】 下午彩排.', 'type': '和平精英', 'owner': '潇男小朋友', 'num': '61.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '[交友]招优质女主持，待遇绝对令你满意！', 'type': '交友', 'owner': 'YAQ一水晶之恋', 'num': '29.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '费电主播 在线挨揍', 'type': '颜值', 'owner': '怡可心呀', 'num': '28.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '健康哥上线！！！', 'type': 'DOTA2', 'owner': 'ZippO宝哥', 'num': '61.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '最近比赛多，疯狂练起来！', 'type': 'QQ飞车', 'owner': '17丶Xtreme', 'num': '60.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '打一天训练赛 准备比赛', 'type': 'LOL手游', 'owner': '心星丶RJ哥', 'num': '60万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新团开播欢迎前来围观', 'type': '舞蹈', 'owner': '恋爱少女团', 'num': '28.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新主播第二天 7209616', 'type': '颜值', 'owner': '奶糕酱o', 'num': '28.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '208114不管你来不来我都在这等你哦', 'type': '炉石传说', 'owner': '恩基爱萌萌哒的狗贼', 'num': '59.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '3小时大师上宗师教学！@', 'type': 'lol云顶之弈', 'owner': 'NB丶条皮兄弟', 'num': '59.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '轮播：音乐区第一长枪', 'type': '永劫无间', 'owner': '衣服ovo', 'num': '59.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '颜值区最软的小柿子双倍亲密度啦', 'type': '颜值', 'owner': '妮妮别吃了', 'num': '27.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '个胶几人呐', 'type': '颜值', 'owner': '米米nnn', 'num': '27.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '郑州创D：618电脑活动进行中', 'type': '数码科技', 'owner': '创D电脑联盟', 'num': '59万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '今天又是喂饭的一天', 'type': '和平精英', 'owner': '逝呆呆', 'num': '58.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【擦擦】上分上分', 'type': 'APEX英雄', 'owner': '擦擦哥哥丶', 'num': '58.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '东北乡村美食，民以食为天，吃货似神仙！', 'type': '美食', 'owner': '百味大彭L', 'num': '27.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '四川小漂亮', 'type': '颜值', 'owner': '半只小月亮o', 'num': '27.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '轰轰：大家一定不要感冒。', 'type': '绝地求生', 'owner': '轰轰仔', 'num': '58.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国一典韦带粉一局不输有车位', 'type': '王者荣耀', 'owner': '小宗阿', 'num': '57.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '热血传奇-创世冰雪--传奇', 'type': '传奇', 'owner': '忘忧妹妹', 'num': '57.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '主播害羞~ 请主动~', 'type': '舞蹈', 'owner': '奶优米呀', 'num': '27.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【交友】招收全职优质女主持', 'type': '交友', 'owner': '爱约一超级巨星', 'num': '27万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '无敌马超教学，专业带粉上分！！！', 'type': '王者荣耀', 'owner': '子夜小呆呆', 'num': '57.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季冲冲王者', 'type': '王者荣耀', 'owner': 'O泡果奶1111', 'num': '57.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '26岁天才少年 9442053', 'type': '穿越火线', 'owner': '白鲨常旭DBQ', 'num': '57.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '早起晨跑的杨颖', 'type': '颜值', 'owner': '青榎', 'num': '26.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '没故事 你要来写吗', 'type': '颜值', 'owner': '十壹y', 'num': '26.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '赛季第一天 带粉速冲王者', 'type': '王者荣耀', 'owner': '解说梦梦', 'num': '57.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '小舅子升学宴，23号见', 'type': '穿越火线', 'owner': '鲜肉赵山河', 'num': '57.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【2021PSI】6月25日', 'type': '和平精英', 'owner': '和平精英官方赛事', 'num': '56.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '很高兴认识你，这是一个靓女的直播间~', 'type': '颜值', 'owner': 'Thea柒清', 'num': '26.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '又青：不渡劫不下播', 'type': '颜值', 'owner': 'Carol又青', 'num': '26.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【PEL国际主播公开赛 】', 'type': '和平精英', 'owner': '皮西蒙丷', 'num': '55.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '大师认真冲分', 'type': '英雄联盟', 'owner': '小康top', 'num': '55.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '阿扎：无挂服顶尖大师局！相声队', 'type': '绝地求生', 'owner': '阿扎y', 'num': '54.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '✨周星驰✨星爷 华语 经典 电影 神乐', 'type': '一起看', 'owner': '进击の神乐', 'num': '26.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '遇到你是我的幸运 感谢阿拉哥 可甜哥宠冠', 'type': '交友', 'owner': 'Yw一治愈仙境', 'num': '26.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '录像：通宵结局看鱼吧', 'type': '欢乐斗地主', 'owner': '天地丶小旺', 'num': '54.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【直播】菁英大奖杯-Day2', 'type': 'LOL手游', 'owner': '斗鱼英雄联盟手游赛事', 'num': '54.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【下午档】职业女边六酱单排教学局', 'type': '王者荣耀', 'owner': 'aGirl六六', 'num': '54.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '贵阳彭于晏教你学做菜', 'type': '美食', 'owner': '拜托了小翔哥', 'num': '26.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '嗨玩-约会圣地/温御聊吧', 'type': '交友', 'owner': '嗨玩一约会圣地', 'num': '26.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【叶繁】传奇4加油，渡劫', 'type': 'CFHD', 'owner': '叶繁i', 'num': '53.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '鱼玄机：今天自信加倍', 'type': '英雄联盟', 'owner': '鱼玄机丶甬娱', 'num': '53.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '备战夏季超级杯 训练一天！', 'type': 'LOL手游', 'owner': '阿紫Yazi', 'num': '53.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【虫哥】魔性吐槽童年回忆', 'type': '原创IP', 'owner': '虫哥说电影', 'num': '26.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '招主持/古典冠/恋爱指南针', 'type': '交友', 'owner': 'YAQ一王者1st', 'num': '26.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服野。射微信区有位置，一把不输', 'type': '王者荣耀', 'owner': '雲小帆', 'num': '53.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【机长】神作直播！刺激！', 'type': '恐怖游戏', 'owner': '机长YG', 'num': '53.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【和平精英微博杯2】每晚18点', 'type': '和平精英', 'owner': '斗鱼和平官方赛事', 'num': '53.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '❤️怡寳❤️ 国民女友陪看', 'type': '一起看', 'owner': '活捉一只小怡寶', 'num': '26万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '永远为忠诚 专一 安全感心动', 'type': '颜值', 'owner': '隔壁网友筱叶', 'num': '26万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '观影游园会-绝地求生', 'type': '绝地求生', 'owner': '绝地求生赛事8', 'num': '52.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '❤空调开好辣，快进来凉快一下吧❤', 'type': '部落冲突', 'owner': '辉辉猪宝宝', 'num': '52.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '25日13点 CFEL 越南', 'type': '穿越火线', 'owner': '穿越火线赛事专用', 'num': '52.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '第一颜值女神厅/最高待遇招收主持', 'type': '交友', 'owner': '爱约一只念初见', 'num': '25.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '✨神乐✨华语 经典 电影 沈腾 周星驰', 'type': '一起看', 'owner': '进击的神乐', 'num': '25.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '观影游园会-和平精英', 'type': '和平精英', 'owner': '和平精英赛事3', 'num': '52.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季五排嘎嘎上星', 'type': '王者荣耀', 'owner': 'RW侠小夜', 'num': '52万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'LDL夏季赛WZvsTT.Y', 'type': '英雄联盟', 'owner': 'LDL官方直播间', 'num': '52万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【周末】港式动作，最新片单', 'type': '原创IP', 'owner': '周末电影院2', 'num': '25.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '武汉长江大桥实时直播', 'type': '社会人文', 'owner': '丝绸之路手机台', 'num': '25.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '有位置 国一那可露露玄策在线带粉丝飞', 'type': '王者荣耀', 'owner': '挽风ac', 'num': '52万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/8008304_1611.png/webpdy1'}
{'title': '国服中路 进来学技术！', 'type': '王者荣耀', 'owner': '阿锟ya', 'num': '51.6万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/3422363_1611.png/webpdy1'}
{'title': 'Elisa 2021夏季邀请赛', 'type': 'CS:GO', 'owner': '斗鱼CSGO赛事频道2', 'num': '51.6万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/roomCove_coverUpdate_2021-06-03_72d1308bd5187e89e440f5b4d5fb52f7.png/webpdy1'}
{'title': '三个野男人的同居生活', 'type': '美食', 'owner': '野居青年', 'num': '25.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【刘老师】每天爆笑解说不断', 'type': '原创IP', 'owner': '刘老师说电影', 'num': '25.2万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/8317926_1606.png/webpdy1'}
{'title': 'Yagao第一视角', 'type': '英雄联盟', 'owner': '英雄联盟赛事活动', 'num': '51.1万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/673320_1611.png/webpdy1'}
{'title': '【技术控御姐】新赛季咯', 'type': '王者荣耀', 'owner': '顾小奈丶', 'num': '51万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/8751050_1612.png/webpdy1'}
{'title': '【三明治】道理简单 继续努力', 'type': 'APEX英雄', 'owner': '一口三明治3Mz', 'num': '50.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '风里雨里-大哥大姐直播间等你', 'type': '户外', 'owner': '追梦的韭菜', 'num': '25万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/9479874_1612.png/webpdy1'}
{'title': '我也喜欢被偏爱', 'type': '颜值', 'owner': '下知子呀', 'num': '24.9万', 'img_src': 'https://rpic.douyucdn.cn/live-cover/appCovers/2021/06/06/6691855_20210606190706_small.jpg/webpdy1'}
{'title': '全英雄联盟最下饭的饭堂', 'type': '英雄联盟', 'owner': '英雄联盟小饭堂', 'num': '50.9万', 'img_src': 'https://rpic.douyucdn.cn/asrpic/210623/9295826_1612.png/webpdy1'}
{'title': '小孩曾卓君：14罚站赛', 'type': '格斗游戏', 'owner': '游天堂', 'num': '50.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '云缨初体验', 'type': '王者荣耀', 'owner': '胡富强r', 'num': '50.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '距离5.20km/上周榜1：点電小可爱', 'type': '交友', 'owner': '争锋一人间值得', 'num': '24.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '野有蔓草，零露漙兮。', 'type': '颜值', 'owner': '漙兮吖', 'num': '24.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'pei彩排~', 'type': '和平精英', 'owner': '林熙Lxxi', 'num': '50.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'UNiboy第一视角', 'type': '英雄联盟', 'owner': '赛事专用直播间2', 'num': '50.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'Geeker go金牌汽车主播帮你选车', 'type': '汽车', 'owner': '九江008', 'num': '50.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '鸣谢“不丢哥”对本厅的宠爱招主持男女都可', 'type': '交友', 'owner': '争锋一梦中城堡', 'num': '24.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '越努力越幸运呀', 'type': '舞蹈', 'owner': 'mystery梦梒', 'num': '24.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服镜赛季初带粉连胜上王者', 'type': '王者荣耀', 'owner': 'Ye诺风', 'num': '50.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '破釜沉舟 不进则退', 'type': '绝地求生', 'owner': '17小鬼丷', 'num': '49.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '双倍亲密度三倍经验值 6264482', 'type': '天天象棋', 'owner': '九妹笑笑', 'num': '48.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '贫民窟的希望女孩~', 'type': '颜值', 'owner': '阿珂珂珂呀', 'num': '24.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '双倍亲密 打打卡 7052226', 'type': '颜值', 'owner': '晚安小酸奶', 'num': '24.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '❤晏安❤做啥都不排队奥❤', 'type': '原神', 'owner': '晏安anan', 'num': '48.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '象棋飞刀王！布局陷阱专家！', 'type': '天天象棋', 'owner': '帽子戏法Chess', 'num': '47.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季教你们上分', 'type': '王者荣耀', 'owner': '你好我叫阳光啊', 'num': '47.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【周末】恐怖上新，经典再现', 'type': '原创IP', 'owner': '周末电影院', 'num': '24.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '等你翻越千万山海 披星戴月而来', 'type': '颜值', 'owner': '小苏打Da', 'num': '24.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '咻咻咻~用快乐为你加速', 'type': '文化', 'owner': '陈翔六点半', 'num': '47.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '欢迎上车 工作使我快乐', 'type': '绝地求生', 'owner': '蜜茶Aimee', 'num': '47.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '一个很柔弱的妹妹~', 'type': '绝地求生', 'owner': '沐沐大人ok', 'num': '47.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '叫上兄弟姐妹，还有狗子一起去挖笋', 'type': '美食', 'owner': '华农兄弟V', 'num': '24.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '想你成为我的守护', 'type': '颜值', 'owner': '帆妹儿超甜', 'num': '24.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '景天：大师冲个第一', 'type': '英雄联盟', 'owner': '景天zxz', 'num': '47万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '成都嘉年华副舞台 欣赏另一桌精彩', 'type': 'JJ斗地主', 'owner': 'JJ比赛副房间', 'num': '46.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '清一色17罗汉杠上开花', 'type': '英雄联盟', 'owner': '杨驴驴y', 'num': '46.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '相遇皆是注定 8744383', 'type': '颜值', 'owner': '千岛寒流q', 'num': '24.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'Dorothy多萝西的直播间', 'type': '美食', 'owner': 'Dorothy多萝西', 'num': '24万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '最新布局陷阱，中局飞刀，残棋组合攻杀', 'type': '天天象棋', 'owner': '八卦象棋大师', 'num': '46.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '峡谷冲冲今天上大师连胜！', 'type': '英雄联盟', 'owner': '清风亚索QAQ', 'num': '46.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '能带你买车的，只有我，实战派。', 'type': '汽车', 'owner': '雪鹰哥麻辣车评', 'num': '46万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '恭喜东游哥喜提国王', 'type': '二次元', 'owner': '颐颐儿', 'num': '24万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '清纯女大学生！', 'type': '颜值', 'owner': '阿梨small', 'num': '23.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'OB黑铁四：诶 我又活过来了', 'type': '英雄联盟', 'owner': '斗鱼丶华仔', 'num': '45.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新人主播球关注 8523981', 'type': '梦幻西游', 'owner': '皇宫扫地僧', 'num': '45.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '《寒霜》打穿提瓦特，深渊补星。', 'type': '原神', 'owner': '小染指o', 'num': '45.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【唱歌】给我一首歌的时间吧', 'type': '音乐', 'owner': '小野绿子midori', 'num': '23.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '所有的最好都不如刚刚好。', 'type': '颜值', 'owner': '佳妮爱吃辣', 'num': '23.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '专业调号规划海岛越战', 'type': '航海王热血航线', 'owner': 'MU丶欢欢', 'num': '45.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'silentBT：醒醒吧', 'type': '绝地求生', 'owner': 'OMGsilentBT', 'num': '45.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '百星国服野王 带粉丝飞', 'type': '王者荣耀', 'owner': '雨晨in', 'num': '45.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '虚拟唱歌主播，走过路过不要错过噢~', 'type': '二次元', 'owner': '一颗小奶桃', 'num': '23.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '可盐可甜小废物', 'type': '户外', 'owner': '你的王者女孩', 'num': '23.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '中单绝活发条冲大师', 'type': 'LOL手游', 'owner': '小师妹Luna', 'num': '45.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '逗B苍 旋风哥中单盖伦无所畏惧', 'type': '英雄联盟', 'owner': '是逗b苍', 'num': '45万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '披萨OB: IG羞/左手/水TES新上', 'type': '英雄联盟', 'owner': '披萨解说', 'num': '44.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '手捏软柿子，嘴添硬茬子', 'type': '户外', 'owner': '王小贝Amy', 'num': '23.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '双倍亲密度啾咪~', 'type': '颜值', 'owner': '梓狸ZIli', 'num': '23.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '凤孔の至暗时刻', 'type': '天涯明月刀', 'owner': '燕徊硯', 'num': '44.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服李白 连胜第一天王者教学', 'type': '王者荣耀', 'owner': '电竞信白丶', 'num': '44.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '今日甜妹瑶瑶教学', 'type': '王者荣耀', 'owner': '玉兔的美梦', 'num': '44.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '由小哥/曦城/白白冠名/招全职兼职主持', 'type': '交友', 'owner': '争锋一你最珍贵', 'num': '23.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '你心目中三国武将谁最厉害？', 'type': '社会人文', 'owner': '易中天', 'num': '23.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '王牌教百榜战神打游戏，坐井观天，指指点点', 'type': '和平精英', 'owner': '战神菠歌', 'num': '44.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '南迦巴瓦的天空', 'type': '科普', 'owner': '在线天文', 'num': '44.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '女野王连胜冲天梯第一~~', 'type': '王者荣耀', 'owner': '辜婉婉', 'num': '44.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '我在富士山下爱你', 'type': '舞蹈', 'owner': '敏敏姐姐V', 'num': '23.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '上海00后舞蹈主播', 'type': '舞蹈', 'owner': 'Abby杨帆', 'num': '23.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【有车位】清纯少女李云龙', 'type': '使命召唤', 'owner': '乔乔乔大王', 'num': '44.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国际服，单排练心态', 'type': 'LOL手游', 'owner': '心星丶星空', 'num': '44万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '隔壁老王来了.~~~！', 'type': 'DOTA2', 'owner': '叁肆叁肆', 'num': '43.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '距离0.1km/招收女主持 待遇从优', 'type': '交友', 'owner': '惠传一白月光', 'num': '23.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'YJ-一程心路：首开公爵厅冠', 'type': '交友', 'owner': '英爵一程心路', 'num': '23.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服伽罗香香99胜率带粉有位置', 'type': '王者荣耀', 'owner': '斑斑QuQ', 'num': '43.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国际服钻石全能中单~鳄鱼~', 'type': 'LOL手游', 'owner': '心星丶木偶', 'num': '43.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '向快乐，冲冲冲……915 6996', 'type': '王者荣耀', 'owner': '我不是周黑鸭', 'num': '43.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '每天只做一件事就是等你！', 'type': '颜值', 'owner': '是你的妙妙', 'num': '23.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【66，酒鬼，58冠】回收各手主持❤', 'type': '开黑', 'owner': 'An丶一生有你i语薇', 'num': '23万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'PUBG气人女主播', 'type': '绝地求生', 'owner': '野生小师妹', 'num': '43.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '我吹过你吹过的晚风。', 'type': '绝地求生', 'owner': '一只很皮的大七', 'num': '42.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '冲击峡谷之巅 绝活阿卡丽', 'type': '英雄联盟', 'owner': '一只阿卡丽灬小表哥', 'num': '42.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【朱一旦】穷的只剩下钱了', 'type': '趣生活', 'owner': '朱一旦的枯燥生活v', 'num': '23万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '我想问问你还活着吗', 'type': '颜值', 'owner': '林奈baby', 'num': '23万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【小葵】58级萌新导师专业讲解', 'type': '原神', 'owner': '不小葵', 'num': '42.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新赛季国服耀韩信冲分跟车随便赢', 'type': '王者荣耀', 'owner': '韩灯泡', 'num': '42.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '青钢影天花板，峡谷之巅', 'type': '英雄联盟', 'owner': '南风青钢影', 'num': '42.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '因为淋过雨所以总想为你撑伞7361260', 'type': '颜值', 'owner': '一坨奶油吖', 'num': '22.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '手游／端游／歌单／助眠／喜欢的歌只唱给你', 'type': '开黑', 'owner': '慧江一卿本佳人', 'num': '22.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '爱是弥天盖地的比雾还浓，虞姬小姐姐带粉哦', 'type': '王者荣耀', 'owner': '小七月Ay', 'num': '42.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【重播】恭喜DYG获得S1冠军', 'type': 'COD手游', 'owner': '使命召唤大师赛', 'num': '42.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '每日欢乐时光 1340947', 'type': '魔兽怀旧服', 'owner': '痴汉万宝路', 'num': '42万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '会唱歌的小姐姐 长相清冷 内心火热', 'type': '颜值', 'owner': '鹿鹿酱in', 'num': '22.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '河南搬砖人', 'type': '颜值', 'owner': '芝士味的李李', 'num': '22.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '带粉2拖3,赛季初上王者,双区双系统~', 'type': '王者荣耀', 'owner': '狗飞丶', 'num': '41.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '【俞白维】竞技魂燃起来了', 'type': '剑网3', 'owner': '俞白维', 'num': '41.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '月底了XDM抬一手吧~', 'type': 'COD手游', 'owner': '你的兔小美', 'num': '41.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '天青色等烟雨而我~~·~~', 'type': '舞蹈', 'owner': '喜喜诶', 'num': '22.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '魔都黄金俱乐部，游戏开始！', 'type': '颜值', 'owner': '赵小南丶南神殿', 'num': '22.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '失踪人口回归 今天打训练赛', 'type': 'LOL手游', 'owner': '心星丶青一', 'num': '41万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': 'Fview Friday 每周聊聊科技新', 'type': '数码科技', 'owner': 'FView爱否科技', 'num': '40.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '请保护好你的牙齿。。', 'type': '魔兽怀旧服', 'owner': '黑欢喜', 'num': '40.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '所爱隔山海 山海亦可平9622507', 'type': '交友', 'owner': '驿动一四季予你', 'num': '22.5万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '“电影之歌·保定站”电影频道融媒体直播', 'type': '社会人文', 'owner': '1905电影网', 'num': '22.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '你爱我我爱你蜜雪冰城甜蜜蜜', 'type': 'APEX英雄', 'owner': '战地记者阿龙', 'num': '40.6万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服第一梦琪玄策带粉包赢', 'type': '王者荣耀', 'owner': '逗哥不是胖子吖', 'num': '40.4万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '昂赛季初冲刺', 'type': '王者荣耀', 'owner': '陈小凡TuT', 'num': '40.3万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '无聊的开箱的直播间', 'type': '趣生活', 'owner': '无聊的开箱', 'num': '22.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '刘能赵四直播专场', 'type': '原创IP', 'owner': '东北爱情大舞台', 'num': '22.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '新发型！用智慧吃鸡，晚点双排！', 'type': '剑网3', 'owner': 'As童话话话', 'num': '40.2万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服野王带粉有位置', 'type': '王者荣耀', 'owner': '听风贼帅', 'num': '40万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '第一橘子 带粉上星', 'type': '王者荣耀', 'owner': '木木爱吃橘子', 'num': '39.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '人上人都吃什么？饿了必看', 'type': '美食', 'owner': '加油小军哥', 'num': '22.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '老爸评测的直播间', 'type': '趣生活', 'owner': '老爸评测', 'num': '22.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '国服最强中路一只猪新赛季冲王者！', 'type': '王者荣耀', 'owner': '伊之助xx', 'num': '39.8万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '潘多拉X 科尔特大黄蜂妲己 源火麒麟', 'type': '穿越火线', 'owner': '主播蹦嚓149435', 'num': '39.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '快乐风男：亚索or永恩', 'type': '英雄联盟', 'owner': '凉风亚索yasuo', 'num': '39.7万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '安静的听歌~ 9739073', 'type': '颜值', 'owner': '何乐乐gkd', 'num': '22.1万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
{'title': '2倍亲密度a', 'type': '舞蹈', 'owner': '看不腻的Abby', 'num': '21.9万', 'img_src': 'https://shark2.douyucdn.cn/front-publish/sdk-file-master/room_list_default@2x_a960c5b.png'}
```

上面代码爬取四个页面后会报错。

