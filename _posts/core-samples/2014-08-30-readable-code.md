---
layout: post
category : book
tagline: "代码即文档"
tags : [ code-sytle ]
---
{% include JB/setup %}



本文主要摘自 《编写可读代码的艺术》(The Art of Readable Code)

###代码即文档

可读性基本原理

    你所写的代码应当让别人理解它所需的时间最小化

####避免空洞的词语选择更专业的词语

get() 很抽象，应该具体指明 get 什么,从哪里get,当然，你如果把这两个作为get
的参数，也是一个好的命名。

size() 就不如 Height(),  NumNodes(), MemoryBytes() 等更具象的词

尽量少使用 tmp, ret，i,j, iter 等空泛的词语，取而代之的是具象的，tmp
具体指什么xxx_tmp，ret 具体返回什么，比如xxx_ret, 循环中具体是循环什么
xxx_i，如果可能尽量指明。

    对比如下代码

    for(int i = 0; i < club.size(), i++)
        for(int j = 0; j < club[i].members.size(), j++)
            for(int k = 0; k < users.size(), k++)
                //这样的bug 识别度较下面要难得多
                //if (club[i].members[k] == user[j]) 
                if (club[i].members[j] == user[k])
                    count << "user[" << j << "] is in club[" << i << i "]" << endl;

    for(int ci = 0; ci < club.size(), ci++)
        for(int mi = 0; mi < club[ci].members.size(), mi++)
            for(int ui = 0; ui < users.size(), ui++)
                if (club[ci].members[mi] == user[ui])
                    count << "user[" << j << "] is in club[" << i << i "]" << endl;

    当然如果你只有一个for循环，那么i,j,k,iter等都是可以接受的。

####让名字携带更多的信息

    比如 string id = "af2d4" //是十六进制的字母，显然 hex_id 更合适

    如下
    start(int delay)       start(int delay_secs)
    sleep(int time)        sleep(int ms)
    data                   data_urlenc

* 如果变量涉及单位，末尾加上单位是一个不错的选择。比如 delay_ms, content_MB
* 如果是 bool 值，增加 is, can, has,should 等为变量前缀，会使得变量意义非常明确, 如 is_password, is_logical等等
* 有一套规范来指导不同类型的变量。比如类名，方法名，属性，私有属性，常量，变量，宏，静态变量等等。
* 经常思考，这个变量别人会理解偏差么？是提高你命名的最有效方法。

   比如 stop(), pause(), kill(), clip(), truncate(), inherit()

    推荐用 begin end 表示不包含最后元素的范围
           first last 表示包含最后的元素范围

* 体味命名后面的意义

    比如 size() countElement() 显然，前面要较后面感觉轻量好多，前面好像数据类型中有length属性，
    你可以O(1)得到长度，而后面，显然是经过遍历O(n)计算后的结果

####合理的布局

    合理布局

		一般比较好的布局
        public class PerformanceTester {
            public static final TcpConnectionSimulator wifi = new
                TcpConnectionSimulator(
                    500, /* Kbps */
                    80, /* millisecs latency */
                    200, /* jitter */
                    1 /* packet loss % */);
            public static final TcpConnectionSimulator t3_fiber =
                new TcpConnectionSimulator(
                    45000, /* Kbps */
                    10, /* millisecs latency */
                    0, /* jitter */
                    0 /* packet loss % */);
            public static final TcpConnectionSimulator cell =
                new TcpConnectionSimulator(
                    100, /* Kbps */
                    400, /* millisecs latency */
                    250, /* jitter */
                    5 /* packet loss % */);
                }

		更简洁、精致的布局
        public class PerformanceTester {
            // TcpConnectionSimulator(throughput, latency, jitter, packet_loss)
            //                          [Kbps]      [ms]    [ms]    [percent]

            public static final TcpConnectionSimulator wifi
                = new TcpConnectionSimulator(500, 80, 200, 1);

            public static final TcpConnectionSimulator t3_fiber
                = new TcpConnectionSimulator(45000, 10, 0, 0);

            public static final TcpConnectionSimulator cell
                = new TcpConnectionSimulator(100, 400, 250, 5)}

    合理重构
	    重构前
		// Turn a partial_name like "Doug Adams" into "Mr. Douglas Adams".
		// If not possible, 'error' is filled with an explanation.
		string ExpandFullName(DatabaseConnection dc, string partial_name, string* error);
		DatabaseConnection database_connection;
		string error;

		assert(ExpandFullName(database_connection, "Doug Adams", &error)
							== "Mr. Douglas Adams");
		assert(error == "");
		assert(ExpandFullName(database_connection, " Jake Brown ", &error)
							== "Mr. Jacob Brown III");
		assert(error == "");
		assert(ExpandFullName(database_connection, "No Such Guy", &error) == "");
		assert(error == "no match found");
		assert(ExpandFullName(database_connection, "John", &error) == "");
		assert(error == "more than one result");

		重构后

		CheckFullName("Doug Adams", "Mr. Douglas Adams", "");
		CheckFullName(" Jake Brown ", "Mr. Jake Brown III", "");
		CheckFullName("No Such Guy", "", "no match found");
		CheckFullName("John", "", "more than one result");

		void CheckFullName(string partial_name,
						   	string expected_full_name,
							string expected_error) {
			// database_connection is now a class member
			string error;
			string full_name = ExpandFullName(database_connection, partial_name, &error);
			assert(error == expected_error);
			assert(full_name == expected_full_name);
		}

	 列对齐

	 	大多数人觉得列对齐不好，是因为，维护复杂，不过建议自己尝试下，也许只是看起来而已。
	 	CheckFullName("Doug Adams" 	, "Mr. Douglas Adams" , "");
		CheckFullName("Jake Brown "	, "Mr. Jake Brown III", "");
		CheckFullName("No Such Guy" , "" 				  , "no match found");
		CheckFullName("John" 		, "" 				  , "more than one result");

		commands[] = {
		...
		{ "timeout" , 		NULL , 				cmd_spec_timeout},
		{ "timestamping" , 	&opt.timestamping , cmd_boolean},
		{ "tries" , 		&opt.ntry , 		cmd_number_inf},
		{ "useproxy" , 		&opt.use_proxy , 	cmd_boolean},
		{ "useragent" , 	NULL , 				cmd_spec_useragent},
		...
		};

	代码按块组织

		class FrontendServer {
			public:
				FrontendServer();
				void ViewProfile(HttpRequest* request);
				void OpenDatabase(string location, string user);
				void SaveProfile(HttpRequest* request);
				string ExtractQueryParam(HttpRequest* request, string param);
				void ReplyOK(HttpRequest* request, string html);
				void FindFriends(HttpRequest* request);
				void ReplyNotFound(HttpRequest* request, string error);
				void CloseDatabase(string location);
				~FrontendServer();
		};

		class FrontendServer {
			public:
				FrontendServer();
				~FrontendServer();

				// Handlers
				void ViewProfile(HttpRequest* request);
				void SaveProfile(HttpRequest* request);
				void FindFriends(HttpRequest* request);

				// Request/Reply Utilities
				string ExtractQueryParam(HttpRequest* request, string param);
				void ReplyOK(HttpRequest* request, string html);
				void ReplyNotFound(HttpRequest* request, string error);

				// Database Helpers
				void OpenDatabase(string location, string user);
				void CloseDatabase(string location);
			};



		# Import the user's email contacts, and match them to users in our system.
		# Then display a list of those users that he/she isn't already friends with.
		def suggest_new_friends(user, email_password):
			friends = user.friends()
			friend_emails = set(f.email for f in friends)
			contacts = import_contacts(user.email, email_password)
			contact_emails = set(c.email for c in contacts)
			non_friend_emails = contact_emails - friend_emails
			suggested_friends = User.objects.select(email__in=non_friend_emails)
			display['user'] = user
			display['friends'] = friends
			display['suggested_friends'] = suggested_friends
			return render("suggested_friends.html", display)


		def suggest_new_friends(user, email_password):
			# Get the user's friends' email addresses.
			friends = user.friends()
			friend_emails = set(f.email for f in friends)

			# Import all email addresses from this user's email account.
			contacts = import_contacts(user.email, email_password)
			contact_emails = set(c.email for c in contacts)

			# Find matching users that they aren't already friends with.
			non_friend_emails = contact_emails - friend_emails
			suggested_friends = User.objects.select(email__in=non_friend_emails)

			# Display these lists on the page. display['user'] = user
			display['friends'] = friends
			display['suggested_friends'] = suggested_friends

			return render("suggested_friends.html", display)

	一致性比风格更重要

	比如
		if () {
		}

		if ()
		{
		}

	还有
		void
		func()
		{}

		void func()
		{}

	等等仅仅是风格问题，但一致性是最重要的。

####注释

	我假定你的代码已经是基本能够自解释的了。

* 不要因为代码命名不好而注释
* 给代码有歧义、陷阱、瑕疵写注释。如TODO, FIXME(无法运行的代码),HACK(对一个问题不得不采用粗暴的办法)，XXX(危险)
* 给关键的常量注释。不是所有的常量需要注释，如果一些参数不建议随意修改，那么应该注释指明，如 
		node_core_ratio = 2;   //当 rate为2时性能最好。
* 对一个接口、文件的总结性进行注释，这样让读者很容易立刻接口或文件主要做了什么。
* 对非常规的做法注释，这可以让维护者了解作者采用这样做法的考虑。再好的代码只能告诉自己是怎么样的，但无法告诉维护
代码的人，为什么这样做。
* 给代码的特殊情况写注释，但这不是必须的。比如，

	split(path) //指明，你对 "/a/b/c/" 和 "/a/b/c" 的处理的区别。

####变量

渐少变量就会减少代码行数，也就使得代码更加简洁。但一味追求减少变量定义并不是可取的。如下的原则可以指导我们更好地判读：

* 没有影响代码的可读性
* 减少冗余变量。
* 减小变量的作用域。
* 尽量渐少变量被再次赋值。

	var setFirstEmptyInput = function (new_value) {
		var found = false;
		var i = 1;
		var elem = document.getElementById('input' + i);
		while (elem !== null) {
			if (elem.value === '') {
				found = true;
				break;
			}
			i++;
			elem = document.getElementById('input' + i);
		}
		if (found) elem.value = new_value;
		return elem;
	};


	var setFirstEmptyInput = function (new_value) {
		for (var i = 1; true; i++) {
			var elem = document.getElementById('input' + i);
			if (elem === null)
				return null; // Search Failed. No empty input found.
			if (elem.value === '') {
				elem.value = new_value;
				return elem;
			}
		}
    };

* 什么是冗余的变量？
	没有拆分任何表达式
	没有更好澄清变量的意义
	只用过一次

	now = datetime.datetime.now()
    root_message.last_view_time = now

	root_message.last_view_time = datetime.datetime.now()

####代码重构

* 为了关注高层实现，如果代码在解决一个子问题，就应该在一个独立的接口中完成。
* 每个一个独立的接口是可测试的
* 某些代码可以独立为基础的工具或库
* 把一个接口的实现过于分散有时候又是对可读性的伤害，必须学会平衡。
* 一个接口只做一件事。

这并不是鼓励分散接口，除非这个独立的功能代码可能被重用或只一个完整的子功能。
如果通过简单的空行能够实现就应该用空行，

完

