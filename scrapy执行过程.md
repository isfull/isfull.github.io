```
1. scrapy crawl chouti --nolog
	
2. 找到 SCHEDULER = "scrapy_redis.scheduler.Scheduler" 配置并实例化调度器对象
	- 执行Scheduler.from_crawler
	- 执行Scheduler.from_settings
		- 读取配置文件：
			SCHEDULER_PERSIST			 # 是否在关闭时候保留原来的调度器和去重记录，True=保留，False=清空
			SCHEDULER_FLUSH_ON_START     # 是否在开始之前清空 调度器和去重记录，True=清空，False=不清空
			SCHEDULER_IDLE_BEFORE_CLOSE  # 去调度器中获取数据时，如果为空，最多等待时间（最后没数据，未获取到）。
		- 读取配置文件：	
			SCHEDULER_QUEUE_KEY			 # %(spider)s:requests
			SCHEDULER_QUEUE_CLASS		 # scrapy_redis.queue.FifoQueue
			SCHEDULER_DUPEFILTER_KEY     # '%(spider)s:dupefilter'
			DUPEFILTER_CLASS			 # 'scrapy_redis.dupefilter.RFPDupeFilter'
			SCHEDULER_SERIALIZER		 # "scrapy_redis.picklecompat"

		- 读取配置文件：
			REDIS_HOST = '140.143.227.206'                            # 主机名
			REDIS_PORT = 8888                                   # 端口
			REDIS_PARAMS  = {'password':'beta'}                                  # Redis连接参数             默认：REDIS_PARAMS = {'socket_timeout': 30,'socket_connect_timeout': 30,'retry_on_timeout': True,'encoding': REDIS_ENCODING,}）
			REDIS_ENCODING = "utf-8"      
	- 示例Scheduler对象
	
3. 爬虫开始执行起始URL
	- 调用 scheduler.enqueue_requests()
		def enqueue_request(self, request):
			# 请求是否需要过滤？
			# 去重规则中是否已经有？（是否已经访问过，如果未访问添加到去重记录中。）
			if not request.dont_filter and self.df.request_seen(request):
				self.df.log(request, self.spider)
				# 已经访问过就不要再访问了
				return False
			
			if self.stats:
				self.stats.inc_value('scheduler/enqueued/redis', spider=self.spider)
			# print('未访问过，添加到调度器', request)
			self.queue.push(request)
			return True
	
4. 下载器去调度器中获取任务，去下载
	
	- 调用 scheduler.next_requests()
		def next_request(self):
			block_pop_timeout = self.idle_before_close
			request = self.queue.pop(block_pop_timeout)
			if request and self.stats:
				self.stats.inc_value('scheduler/dequeued/redis', spider=self.spider)
			return request
```
