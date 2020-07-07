# music-top-recommend

1.	对三个数据进行预处理，合并用户与物品相关信息，数据字段包含itemid、userid、用户信息(年龄、性别、收入、地区)、物品信息（名字、描述、时长、标签）、用户行为数据(收听时长)等。
2.	粗排召回阶段使用CB算法，基于内容进行jieba中文分词，计算itemid对应分词的tfidf分数，整理训练数据；使用mr 协同
过滤进行相关性计算，训练得到物品之间对应分数item-item；CF算法则通过协同过滤将UI矩阵转成II矩阵，格式化数据后将结果按k/v形式批量灌入redis数据库。
3.	精排阶段利用LR进行推荐排序，得到权重w、b用于模型构建。结合用户与物品标签获取用户与物品特征训练数据。
4.	推荐流程阶段加载特征数据及排序模型，检索redis数据库获取候选集，利用逻辑回归sigmoid函数打分并排序，最终利用可视化页面实现itemid->name进行top10评分相关推荐。

一、数据预处理
1、准备数据：
用户画像数据（user_profile.data）
00ea9a2fe9c6810aab440c4d8c050000,女,26-35,20000-100000,江苏
01a0ae50fd4b9ef6ed04c22a7e421000,女,36-45,0-2000,河北
002db7d2360562dd16828c4b91402000,女,46-100,5000-10000,云南
006a184749e3b3eb83e9eb516d522000,男,36-45,2000-5000,天津
00de61c1d635ad964eef2aefa8292000,女,19-25,2000-5000,内蒙古
字段：userid, gender, age, salary, location

	物品元数据（music_meta）
029900100  徐颢菲《猫的借口》284 国内
字段：itemid, name, desc, total_timelen, location, tags

	用户行为数据（user_watch_pref.sml）
01e069ed67600f1914e64c0fe773094440903091011519
01d86fc1401b283d5828c293be290e0861928091017512
002f4b9c49be9a0b2c13e1c3c4f6a21c891510910138518

二、召回阶段（粗排）：
1、CB算法——候选--gen_cb_train.py
物品元数据：name,desc,tag
经过数据预处理，得到如下格式的cb训练数据：
哲,4090309101,0.896607562717
大连,4090309101,0.568628215367
舞曲,4090309101,0.713898826298
大美妞,4090309101,0.896607562717
网络,4090309101,0.465710816584
伤感,4090309101,0.628141853463
2.协同过滤算法（求相似度的II矩阵）--mr.py
3.批量灌入redis数据库--gen_reclist.py

三、推荐排序LR--rankmodel
1.求w,b构建模型--lr.py
2.解析标签获取用户与物品训练数据-- gen_sampes.py

四、推荐流程阶段--main.py
