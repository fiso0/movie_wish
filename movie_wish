"""
从豆瓣电影获取所有“想看”的电影，然后自动到BT电影天堂去搜索，查看是否有高清资源，最后给出结果
"""

from bs4 import BeautifulSoup
import requests
import re

BT_URL = 'http://www.bttiantang.com'


def get_soup_of(url):
	"""获取某个url的soup
	:param url: 待获取的网页地址
	"""
	html = requests.get(url).text
	soup = BeautifulSoup(html,'lxml')
	return (soup)


def get_bt_search_url(movie_title):
	"""拼接得到在BT天堂搜索某部电影时用到的url"""
	search_head = 'http://www.bttiantang.com/category.php?/'
	search_tail = '/'
	return search_head + movie_title + search_tail


def get_douban_url(url):
	"""根据BT给出的跳转链接获得豆瓣页url
	:param url: BT天堂给出的豆瓣页中间跳转页面地址
	"""
	soup = get_soup_of(url)
	text = soup.get_text()

	try:
		link = re.search("\"([^\"]*)\"", text).group(1)
	except:  # 有可能显示"error!
		link = url
	return link


def search_bt_movie(movie_title):
	"""在BT天堂上搜索电影
	:param movie_title: 需要搜索的电影标题
	:return: 所有搜索结果的BT页面地址和对应的豆瓣页面地址（元祖列表）
	"""
	print('开始搜索BT电影天堂')
	bt_search_url = get_bt_search_url(movie_title)
	soup = get_soup_of(bt_search_url)

	# BT天堂搜索结果的详细电影页面地址
	movies_bt_url = [BT_URL + a.get('href') for a in soup.select('.title .tt a')]

	# BT天堂搜索结果对应的豆瓣页地址
	bt_douban_link = soup.find_all('a', title='去豆瓣查看影片介绍')
	bt_douban_jump = [BT_URL + link.get('href') for link in bt_douban_link]
	bt_douban_url = [get_douban_url(jump) for jump in bt_douban_jump]

	results = [(bt, douban) for bt in movies_bt_url for douban in bt_douban_url]
	return results

def get_downloads(url,file_size=3):
	"""获取某电影下载资源并按大小排序
	:param file_size: 高清电影资源的大小下限，默认3GB
	:param url: BT天堂网址中某部电影的页面地址
	:return: 所有下载资源的大小和地址（字典列表）
	"""
	print(url)
	soup = get_soup_of(url)

	resource = []  # 存放所有满足大小要求的资源
	downloads = soup.find_all('div', class_='tinfo')

	if len(downloads) == 0:
		print('找不到下载资源')
		return []

	for i in downloads:

		if 'xunleigang' in i.text:
			print('资源被网站删除')
			return []

		size = i.p.em.get_text()

		try:
			(size_num, size_unit) = re.search(r'(\d*.?\d*)([GgMm][Bb])', size).groups()
			size_num = float(size_num)
		except AttributeError:
			(size_num, size_unit) = (0, '')

		if (size_unit == 'GB' or size_unit == 'gb') and size_num > file_size:
			down_addr = BT_URL + i.a.get('href')
			down_info = {'size': size_num, 'download_addr': down_addr}
			resource.append(down_info)
			print('\t' + str(size) + '\t下载地址：' + down_addr)
		else:
			print('\t' + str(size) + '\t该资源太小')
	# if len(resource) > 1:
	# 	resource.sort(key=lambda x: x[0])
	return resource


# 用于保存所有结果的列表
movie_list = []

# 豆瓣电影 我的想看 页面
douban_url = 'https://movie.douban.com/people/tsangtszching/wish'
soup = get_soup_of(douban_url)

# 总的想看电影数量
wish_count = int(re.findall(r'\d+', soup.select_one('#db-usr-profile .info h1').text)[0])

# 对每一页解析
print (wish_count)

for start in range(0, wish_count, 15):
	douban_page_url = douban_url + '?start=' + str(start)
	soup = get_soup_of(douban_page_url)

	# 该页内所有的电影条目（默认最大15条）
	list_items = soup.select('.item')

	# 对一部电影解析
	for item in list_items:
		# 保存一部电影的全部信息：包括电影标题，豆瓣链接，下载资源（字典）
		movie = {}

		# 电影标题
		movie_titles = item.select_one('.title').text.replace(' ', '').replace('\n', '').split('/')
		movie['title'] = movie_titles
		print('电影：' + movie_titles[0])

		# 豆瓣电影页网址
		movie_douban_url = item.select_one('.title a').get('href')
		movie['douban_url'] = movie_douban_url
		print('豆瓣链接：' + movie_douban_url)

		# 尝试到BT天堂上搜索该电影，获取搜索结果的BT页面地址和对应的豆瓣页面地址
		search_results = search_bt_movie(movie_titles[0])

		# 根据豆瓣页面是否相同确认为正确的搜索结果
		for (bt, douban) in search_results:
			if douban.split(':')[1] == movie_douban_url.split(':')[1]:
				print('搜索到该电影，开始搜索下载资源')
				# 获取高清资源大小和下载地址（字典列表）
				movie_downloads = get_downloads(bt)
				break
			# else:
			# 	print(douban)
			# 	print(movie_douban_url)
		else:
			print('没有搜索到该电影')
			movie_downloads = []

		# 电影下载资源
		movie['download'] = movie_downloads

		# 将此部电影所有相关信息添加到电影结果列表中
		movie_list.append(movie)

print(movie_list)
