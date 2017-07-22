图书推荐与管理系统(Qmazon)
=============

简介
-------------
这是本人于本科二年级时修读的"面向对象的程序设计(C++)"的课程作业。该系统实现了一个关于图书的评论与推荐系统，类似亚马逊、当当与豆瓣。该系统使用C++作为编程语言，并使用了Qt程序开发框架完成了程序的可视化，搭建了类似PC版QQ风格的界面，具有很高的美观度。本系统针对图书推荐这一核心功能采用了基于用户的协同过滤算法，并结合基于人口统计学的冷启动推荐算法完成了推荐模块的设计。

数据集 ：http://www2.informatik.uni-freiburg.de/~cziegler/BX/

系统展示图
-------------
</br>
![Alt text](http://i.imgur.com/eQuoHYh.png)
![Alt text](http://i.imgur.com/QJBUnHO.png)

需求分析
-------------
此系统对我们提出的需求可以分为前端功能和后端功能，前端功能即普通用户的一些操作功能，包括用户的注册、登录、获取推荐、打分、评论、更改信息、注销账户、查找图书、查找好友等功能，后端功能包括管理员的一些操作功能，包括管理员的登录、添加图书、删除图书、更改图书信息、添加新用户、删除用户、更改用户资料、查找图书、查找用户、注销账户等功能。因此在CLI（命令行界面）下我们需要首先设置登录与注册功能、登陆后根据登录身份的不同来设计可以进行的不同功能。而在GUI（图形用户界面）下我们需要设置登录界面、注册界面、管理员功能界面、用户功能界面、以及用户的各个子功能界面。命令行界面的实现将通过VS、Dev-Cpp来实现，而图形化界面将在CLI代码的基础上通过Qt Creator编译并实现。 

类的设计
------------- 
本程序主要的类有两个，其定义在base.h头文件中：图书(book)类与用户(user)类。具体定义及操作可见base.h头文件，这里便不再多说。其函数具体定义在Main.cpp文件中。

文件操作
-------------

#### 1、图书信息
使用 ifstream打开文件后，将每条读入的数据暂存到一个Book类型的newbook中，这本书的所有数据读完之后，将newbook使用
```c++
books.insert(map<string, Book>::value_type(IS, newbook));
```
加入到总的book这个map类型的对象中中。其中IS为newbook的ISBN码，直接用来索引图书。
书的各类信息分隔使用
```c++
getline(file, value, ';');
IS = string(value, 1, value.length() - 2);
```
表示读到“；”之前的数据，并且IS的内容是value去掉最前面的“与最后面的”所得。
最后使用```c++file.close();```关闭文件。
#### 2、用户信息
与图书信息的读入过程类似。
#### 3、评分信息
先用ifstream打开文件，之后用
```c++
getline(file, value, ';');
string(value, 1, value.length() - 2);
```

读入IS与ID，再直接利用map的索引将信息读入每个信息的对应项之中。并且增加读过这个book的用户信息，与这个user看过的图书信息。
```c++
users[ID].booksRead.insert(map<string, int>::value_type(IS, rank));
books[IS].usersRead.insert(map<string, int>::value_type(ID, rank));
```
最后使用file.close();关闭文件。并且读完用户全部评分信息后，使用迭代器将每个book与每个user的平均评分求出。例下面就是求出每个book的平均分的代码。
```c++	
map<string, Book>::iterator iter = books.begin();
	for (; iter != books.end(); iter++)
	{
		iter->second.aveRank = iter->second.sum / iter->second.cnt;
	}
```
#### 4、文件回写
由于又增加修改删除的图书用户等，在使用过一遍系统后要将所有信息全部重新写入。
#####book文件写回
先用ofstream打开文件，之后先写回标题栏。
```c++ 
file << "\"ISBN\";\"Book-Title\";\"Book-Author\";\"Year-Of-Publication\";\"Publisher\""<< ";\"Image-URL-S\";\"Image-URL-M\";\"Image-URL-L\"" << endl;
```
之后将每一类信息写回，双引号使用\”表示。
最后```c++file.close();```关闭文件。
#####user文件写回。
先用ofstream打开文件，之后先写回标题栏。
	```c++file2 << "\"User-ID\";\"Location\";\"Age\"" << endl;```
之后将每一类信息写回，双引号使用\”表示。
最后```c++file.close();```关闭文件。
#####rank文件写回。先用ofstream打开文件，之后先写回标题栏。
```c++	file3 << "\"User-ID\";\"ISBN\";\"Book-Rating\"" << endl;```
之后将每一类信息写回，双引号使用\”表示。
最后```c++file.close();```关闭文件。

基于用户的协同过滤推荐算法
-------------
对于一个已经读过几本书的用户，我们采用的推荐方法便是采用正常的基于用户的协同过滤算法。基于用户的协同过滤推荐算法的基本思想便是：首先依据依据用户对物品的评价计算出所有用户之间的相似度，之后选出与当前用户最相似的N个用户，再用N个邻居用户对物品的评分，预测当前用户对没有浏览过的物品的可能评分，最后按照预测出的可能评分的高低向当前用户推荐物品。 接下来将对算法的实现进行详细介绍。</br>算法在程序中的位置为recommendsystem1.cpp中的
```
c++User:: getrecommendation()
```
中。首先我们要先进行对与所有用户的相似度计算。计算的基本公式如下：
 
![alt](http://i.imgur.com/wBY7J35.png)
</br>当我们计算当前用户A与用户B的相似度时，首先我们要先去遍历A读过的书，从这些书里去找到B同样也读过的书，找到相应的书后，这本书便是公式中的p。以上公式可分为三个求和部分，因此函数中我采用了sumup,sumdown1,sumdown2分别累加。之后通过套用公式并对所有其共同读过的书进行累加计算出用户A与B的相似度，并通过第一层类似地计算出A与所有用户的相似度，将相似度存于一个
```c++map<string, double>Sims```中，即当前用户与id为String的用户的相似度。</br>
到这里我们计算出了当前用户与所有用户的相似度，接下来我们便可以选取邻居用户进行预测分数。在这里的邻居用户数我选定为全体用户（27w+），原因是由于数据集本来就是就很稀疏本来能够拥有相似度的用户就很少，因此采用全体用户即可最为准确的进行推荐且不会造成速度上的缓慢。预测公式如下：![](http://i.imgur.com/FyhfoNd.png)
   首先我们从所有书中去遍历，并筛选出该用户没有读过的书，假定现在我们要预测图书P的预测评分，则接下来去从所有读过P的人中去寻找，找到与当前用户有相似度的用户，按照上边的公式进行计算并累加，便可以计算出P的预测评分。类似地，便可以计算出所有图书的预测评分。在这里我将所有预测评分存在了一个map<string, double> RankPre中，string指的是书的ISBN码，double指评分。</br>
  之后便是排序过程，这里我首先写了一个将map中的元素按降序排列并存储在Vector的函数```c++void sortMapByValue(map<string, double>& tMap, vector<pair<string, double> >& tVector)```，将之前的Rankpre排序后存于Rvector中，接下来便可以将Rvector前几个元素输出即为推荐图书了。</br>
  在计算推荐图书的过程实际上也顺便进行了推荐好友的计算，即相似度Sims已被我们存储。之后再将Sims调用sortMapByValue函数进行降序排列并存于Svector中，再输出前几个元素便为推荐好友。</br>
  同时，还有一点值得注意。如果一个用户读过书，但读的书过于少或过于“冷门”，以至于没有人与其读过相同的书，即没有人和他拥有相似度，则对他仍作冷启动处理。</br>
  以上便是算法的原理与实现的基本内容。还有值得一提的地方，便是如果两个用户读过的书一模一样，直观上来说其相似度应该是非常大的（1），但套用公式分母则是0，无法计算，因此我采用了特殊处理，如果两个用户读过的书完全一样（isequal），则相似度直接置1，这样或多或少能够降低误差考虑了极端情况。


Qt框架的使用
-------------
本项目使用了Qt进行了C++的可视化。在Main.cpp函数中，首先要将Qt自动产生的各个界面的头文件引入。之后对每个界面每个按钮的事件进行函数书写即可。如删除图书的事件触发函数：
```c++
void menu_admin::on_pushButton_3_clicked()
{
    bool ok;
    // 获取字符串
    QString qid = QInputDialog::getText(this,tr("input"),
              tr("input the id:"),QLineEdit::Normal,tr("admin"),&ok);
    string id=qid.toStdString();
    if(books.find(id)!=books.end()){
        books[id].deletebook();
    }
    else{
        QMessageBox::information(this,tr("tip"),
                         tr("cannot find so you cannot delete!"),QMessageBox::Ok);
    }
}
```
本项目一大亮点之一便是出色的ui设计。而ui设计的完成很大一方面也要归功于Qt这一框架强大的ui设计功能。我们在Qt的ui模式下可视化操作设计过程并通过qss文件进行辅助，而qss文件的语法与css是极其相似的，因此也极易上手。

时空分析
-------------
#### 1、时间复杂度
本程序主要的用的容器是map。在C++中，map根据key的值去查找，本质上讲使用的是红黑树的数据结构，因此搜索时的时间复杂度为O(logn)。在map插入一个键值对时，因为没有最优位置暗示，所以插入删除的时间复杂度为 O(logn)，而插入删除多个键值对的时间复杂度为O(nlogn)，但由于图书用户信息的修改只是有限的几次，故可以直接认为时间复杂度为O(logn)。

在文件读取时，就已求出书的平均分并存好数据，这里时间复杂度为O(n)。文件写回时也是一次性顺序写回，时间复杂度也为O(n)。

修改图书、用户信息，用户评分的时间复杂度都是O(1)。

冷启动推荐算法，将用户按照信息相似度匹配，时间复杂度为O(n)，依次取出用户高评分书籍或者根据200个最相似用户的评分预测当前用户评分，时间复杂度为O(1)，因为200个用户相对于总共20万用户来说可看作一个小的常数。

基于用户的协同过滤算法，为指定用户推荐图书时，先找出与当前用户相似的用户，时间复杂度为O(n),再按照协同过滤算法公式预测计算，由于每个用户读过的书数量不会太多且固定，所以可以看做常数，时间复杂度为O(n)。

综上，本实验的时间复杂度为时间复杂度为O(n)。

#### 2、空间复杂度
本程序的存储思想类似于稀疏矩阵，并不是建立一个N*M的用户 — 图书二维数组，而是只保存有意义的信息值。尽管同一项键值对保存了两次（每种图书被哪些用户看过及评分，每个用户看过哪本图书及评分），但这并不影响最终的空间复杂度，相当于最终空间复杂度为O(nm)。且map容器的空间复杂度为O(n)与O(m)。所以最终程序的空间复杂度为O(nm)。


未来展望与改进
-------------
由于程序采用文件操作，而本项目的数据集较大，三个csv文件共计118MB，故程序入口处的文件读取操作是耗时且用户体验较差的。由于二年级上学期未学习数据库的相关课程，因此没有采用数据库存储，这也是一大遗憾。相信本项目若是之后能采用数据库存储数据，必然能够很大程度上增进用户体验。

