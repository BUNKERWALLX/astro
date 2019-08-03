# clonex
# Dependencias pip install beautifulsoup4
# Antes de arrancar el script instalar pip install beautifulsoup4

# coding: utf-8
# Coded with love and bugs by Abdulraheem Khaled @abdulrah33mk

import os
from re import search
from requests import get
from threading import Thread
from bs4 import BeautifulSoup
from datetime import datetime


# Adding some colors to the script
end = '\033[1;m'
red = '\033[1;31m'
white = '\033[1;37m'
green = '\033[1;32m'


class Tool:  # This class is responsible about Tools
	list = []
	toolNum = num = 1
	toolsFile = 'ToolsList.txt'
	htmlFile = 'ToolsList.html'
	access_token = 'b1ff11362c57b70864100848ef13cf58231ec0a7'  # GitHub API (You can use your access_token)

	def __init__(self, url, *add):  # Constructor
		url = self.getUrl(url)
		u = url[19:].split('/')
		self.author = u[0]
		self.name = u[1]
		self.url = url
		self.num = Tool.toolNum if add else Tool.num
		self.available = self.check(self.url) # Check that the tool is available
		self.desc = self.getDescription() if self.available else None
		if add:  # If user wants to add the tool to the list
			if self.available:
				self.isInstalled = self.exists(self.name)
				self.lastInstall = self.lastInstall() if self.isInstalled else "Couldn't retrieve the date"
				self.lastUpdate = self.lastUpdate()
				self.isUpToDate = type(self.lastInstall) != str and self.lastInstall >= self.lastUpdate
			Tool.toolNum += 1
		else:
			Tool.num += 1

	def getDescription(self):  # Returns tool description
		d = get('https://api.github.com/repos' + self.url[18:] + '?access_token=' + Tool.access_token).json()['description']
		return 'No hay descripcion!' if d is None else d

	def lastUpdate(self):  # Returns last update for the tool on GitHub
		u = get('https://api.github.com/repos' + self.url[18:] + '?access_token=' + Tool.access_token).json()['pushed_at']
		return self.strpTime(str(u.replace('T', ' ')[:-1]))

	def lastInstall(self):  # Returns last installation for the tool on PC
		if self.exists(self.name + '/install'):
			return self.strpTime(open(self.name + '/install', 'r').read())
		else:
			return "Couldn't retrieve the date"

	def clone(self, *path):  # Clone the tool to the path argument
		print 'Installing: ' + self.name + ': ',
		if not os.system('git clone -q ' + self.url + ' ' + ('/tmp/' if path else '') + self.name):
			print green + 'Ok' + end
			open(('/tmp/' if path else '') + self.name + '/install', 'w').write(Tool.strfTime(datetime.now()))
		else:
			print red + 'Error' + end

	def remove(self, *path):  # Delete the passed directory
		if Tool.exists(('/tmp/' if path else '' + self.name)):
			if path:
				print 'Eliminabdo herramienta de tmp : ',
			else:
				print 'Eliminando version anterior of ' + self.name + ': ',
			if not os.system('rm -rf ' + ('/tmp/' if path else '') + self.name):
				print green + 'Ok' + end
			else:
				print red + 'Error' + end

	def copy(self):  # Copy installed tool from tmp to current directory
		print 'Copiando herramienta : ',
		if not os.system('cp -af /tmp/' + self.name + ' ./'):
			print green + 'Ok' + end
		else:
			print red + 'Error' + end

	def printInfo(self):  # Print some information about found tools
		print 'Numero de herramienta: ' + red + str(self.num) + end
		print 'Herramienta: ' + red + self.name + end
		print 'Autor: ' + red + self.author + end
		print 'URL: '+ red + self.url + end
		print 'Disponibilidad: '+ ((green + 'Disponible') if self.available else (red + 'No disponible')) + end
		if self.available:
			print 'Descriction: ' + red + self.desc + end
		if hasattr(self, 'isInstalled'):
			print 'Last Update On GitHub: ' + red + Tool.strfTime(self.lastUpdate) + end
			print 'Last Update On PC: ' + red + (self.lastInstall if type(self.lastInstall) == str else Tool.strfTime(self.lastInstall)) + end
			print 'Actualizar: ' + (green if self.isUpToDate else red) + str(self.isUpToDate) + end
			print 'Estado: ' + (green + 'Instalado' if self.isInstalled else red + 'No Instalado') + end
		print white + '================================================================================' + end

	# --------------
	# Static Methods
	# --------------
	@staticmethod
	def getUrl(url):
		url = search('https:\/\/github\.com(\/\w+([-._]?\w*)+){2}', url.lower())
		if url:
			url = str(url.group())
			for x in ['.git', '/']: url = url[:None if not url.endswith(x) else -len(x)]
			return url

	@staticmethod
	def exists(path):  # Check if path exists or not
		return os.path.exists(path)

	@staticmethod
	def deleteFile(path):  # Delete the past path
		if Tool.exists(path):
			os.remove(path)

	@staticmethod
	def check(url):  # Checks that the tool is available on GitHub
		return get('https://api.github.com/repos' + url[18:] + '?access_token=' + Tool.access_token).ok

	@staticmethod
	def strpTime(date):  # Converts string to datetime object
		return datetime.strptime(date, '%Y-%m-%d %H:%M:%S')

	@staticmethod
	def strfTime(date):  # Converts datetime object to formated string
		return datetime.strftime(date, '%Y-%m-%d %H:%M:%S')

	@staticmethod
	def add(url):  # Add tool using URL
		try:
			url = Tool.getUrl(url)
			if not url or not Tool.check(url):
				raise Exception('La herramienta no está disponible, verifique la URL de la herramienta.\nIt must be like this: https://github.com/User/Tool')
			elif Tool.found(url):
				raise Exception('La herramienta ya se encontró en la lista')
			resource.write(url + '\n')
			print green + 'Nueva herramienta añadida' + end
		except Exception, err:
			print red + str(err) + end

	@staticmethod
	def find(tool, t):  # Search for a tool on GitHub
		if t == 1:
			req = get('https://api.github.com/search/repositories?q=' + tool + '&access_token=' + Tool.access_token)
		else:
			req = get('https://api.github.com/users/' + tool + '/repos?access_token=' + Tool.access_token)
		js = req.json()
		if req.ok:
			foundTools = []
			try:
				numOfTools = input('Encontre {0}{1}{2} herramientas cuantas quieres que te muestre?{0}:{2} '.format(red, str(
					(js['total_count'] if t == 1 else len(js))), end))
			except NameError:
				print red + 'Mala desicion!' + end
				return
			j = js['items'] if t == 1 else js
			for t in j[:numOfTools]:  # Shows the found tools in the found pages
				foundTools.append(t['html_url'])
				Tool(foundTools[-1]).printInfo()
			try:
				choice = input('Cual quieres {0}:{1} '.format(red, end))
			except NameError:
				print red + 'Mala desicion!' + end
				return
			if choice <= len(foundTools):
				wait()
				Tool.add(foundTools[int(choice) - 1])
			else:
				print red + 'elije 1 al ' + str(len(foundTools)) + end

	# --------------
	# Class Methods
	# --------------
	@classmethod
	def reset(cls):  # Reset tools' number
		cls.toolNum = cls.num = 1

	@classmethod
	def found(cls, toolURL):  # Check if the tool was added before
		return toolURL in [tool.url for tool in cls.list]

	@classmethod
	def display(cls):  # Display users tools
		if len(cls.list) == 0:  # Checks if the list is empty or not
			print red + 'No se an agregado nuevas herramientas' + end
		else:
			for tool in cls.list:
				tool.printInfo()

	@classmethod
	def update(cls):  # Update all tools on the list
		try:
			print '[{0}1{1}] Actualiza todas las herramientas\n[{0}2{1}] actualiza viejas herramientas'.format(red, end)
			x = input('Elije del  1 or 2{0}:{1} '.format(red, end))
		except NameError:
			print red + 'Mala desicion!' + end
			return
		if x == 1:
			listToUpdate = cls.list
		elif x == 2:
			listToUpdate = [tool for tool in cls.list if tool.available and not tool.isUpToDate]
		else:
			print red + 'Mala desicion!' + end
			return
		for tool in listToUpdate:
			print green + '\n[' + tool.name + ']' + end
			tool.remove('/tmp/')
			tool.clone('/tmp/')
			tool.copy()
			tool.remove('/tmp/')
			print white + '================================================' + end
		print green + '\nTodas las herramientas han sido actualizadas' + end

	@classmethod
	def reinstall(cls):  # Reinstall all tools on the list
		for tool in cls.list:
			print green + '\n[' + tool.name + ']' + end
			tool.remove()
			tool.clone()
			print red + '================================================' + end
		print green + '\nTodas las herramientas han sido reinstaladas' + end

	@classmethod
	def importToolsHtml(cls):  # Import tools from HTML page
		if cls.exists(cls.htmlFile):
			for tool in BeautifulSoup(open(cls.htmlFile, 'r').read(), 'html.parser').find_all('a'):
				cls.add(tool['href'])

	@classmethod
	def importToolsDir(cls):  # Import tools from current directory
		directories = [d for d in os.listdir('./') if os.path.isdir(d) and cls.exists(d + '/.git/config')]
		for d in directories:
			x = search('https://github.com/.+\w+', open(d + '/.git/config').read())
			if x:
				cls.add(x.group())

	@classmethod
	def exportTools(cls):  # Export tools on the list to HTML page
		Tool.deleteFile(cls.htmlFile)
		#   Beginning
		htmlCode = """<html><head><title>clonex</title><style>@font-face{font-bunker;src:url(https://bunkerwallx.com);}
body{color:red;background-color:black;text-align:center;font-size:25px;font-family:Hacked}
h1{font-size:55px;}a{display:block;width:200px;color:white;background-color:red;text-decoration: none;
border: 1px solid red;border-radius:10px;margin:20px auto;padding:5px;}</style></head><body><h1>clonex</h1>"""

		# Middle
		htmlCode += "".join(["<a href=" + tool.url + ">" + tool.name + "</a>" for tool in cls.list])

		# End
		htmlCode += "./Abdulraheem_Khaled</body></html>"
		soup = BeautifulSoup(htmlCode, 'html.parser')
		open(cls.htmlFile, 'w').write(soup.prettify())
		print 'Las herramientas se han exportado a ' + red + cls.htmlFile + end


def wait():  # Wait until all tools on the list are updated
	if not updated:
		print red + 'Por favor espere para actualizar la lista de herramientas' + end
	while not updated:
		pass


def update():  # Update the list of tools
	global updated
	Tool.list =[Tool(tool, True) for tool in set([Tool.getUrl(tool.strip()) for tool in resource])]
	updated = True


if __name__ == '__main__':  # Main method
	while True:
		Tool.reset()  # Reset number of tools
		resource = open(Tool.toolsFile, 'a+')  # List of added tools
		resource.seek(0)  # Start reading from the beginning of file
		updated = False  # Check that tools list has been read
		Thread(target=update).start()  # Using threading to
		os.system('clear')
		print """{1}
**********************************************************************************
                                                                              
oooooooooo.                          oooo                           
`888'   `Y8b                         `888                           
 888     888 oooo  oooo  ooo. .oo.    888  oooo   .ooooo.  oooo d8b 
 888oooo888' `888  `888  `888P"Y88b   888 .8P'   d88' `88b `888""8P 
 888    `88b  888   888   888   888   888888.    888ooo888  888     
 888    .88P  888   888   888   888   888 `88b.  888    .o  888     
o888bood8P'   `V88V"V8P' o888o o888o o888o o888o `Y8bod8P' d888b    
                                                                              

	  				./{1}bunkerwallx.com {0}@{1}bunkerwallx{1}
********************************************************************************{2}
[{0}A{2}] agrega una herramienta de una URL
[{0}F{2}] busca una herramienta de GitHub
[{0}R{2}] reinstalla tus herramentas
[{0}U{2}] actualiza las herramientas
[{0}S{2}] ver tus herramientas
[{0}D{2}] eliminar lista de herramientas
[{0}X{2}] exporta herramientas de HTML
[{0}M{2}] Importa  tus herramientas
[{0}E{2}] Salida """.format(red, white, end)
		choice = raw_input('que chingados quieres?' + red + ': ' + end).lower()
		# ---------------------------------------------------------------------------------
		if choice == 'a':
			print '[' + red + "agrega una herramienta" + end + ']'
			r = "y"  # Add more tools
			while r == "y":
				wait()
				Tool.add(raw_input("ingresa direccion GitHub " + red + ": " + end).lower())
				r = raw_input("Agrega nueva herramienta ({0}Y{1} or {0}N{1}): ".format(red, end)).lower()
		# ---------------------------------------------------------------------------------
		elif choice == 's':
			print '[' + red + "ver tus herramientas" + end + ']'
			wait()
			Tool.display()
		# ---------------------------------------------------------------------------------
		elif choice == 'r':
			print '[' + red + "Reinstalar herramientas" + end + ']'
			wait()
			Tool.reinstall()
		# ---------------------------------------------------------------------------------
		elif choice == 'u':
			print '[' + red + "actualiza herramientas" + end + ']'
			wait()
			Tool.update()
		# ---------------------------------------------------------------------------------
		elif choice == 'f':
			print '[' + red + "busca herramientas" + end + ']'
			print '[{0}1{1}] busca usando nombre\n[{0}2{1}] busca usando usuario de github'.format(red, end)
			try:
				choice = input('elije uno{0}:{1} '.format(red, end))
			except NameError:
				choice = 3
			if choice == 1 or choice == 2:
				Tool.find(raw_input('Ingresa el nombre qe buscas' + red + ': ' + end), choice)
			else:
				print red + 'Mala desicion!' + end
		# ---------------------------------------------------------------------------------
		elif choice == 'd':
			print '[' + red + 'elimina tu herramienta' + end + ']'
			wait()
			Tool.deleteFile(Tool.toolsFile)
			print green + 'eliminar herramientas' + end
		# ---------------------------------------------------------------------------------
		elif choice == 'm':  # Importing your tools
			print '[' + red + 'Importa herramientas' + end + ']'
			wait()
			print 'tienes dos opciones{0}:{1}\n[{0}1{1}] Importa a  HTML\n[{0}2{1}] Importa a directorio '.format(red, end)
			try:
				choice = input('elije una{0}:{1} '.format(red, end))
			except NameError:
				choice = 3
			if choice == 1:  # HTML
				Tool.importToolsHtml()
			elif choice == 2:  # Current directory
				Tool.importToolsDir()
			else:
				print red + 'mala desicion!' + end
		# ---------------------------------------------------------------------------------
		elif choice == 'x':  # Exporting your tools
			print '[' + red + 'Exportando  a HTML' + end + ']'
			wait()
			Tool.exportTools()
		# ---------------------------------------------------------------------------------
		elif choice == 'e':  # Exit
			print white + 'adios  ...que chingue su madre donald trump junto con peña nieto par de homoxesuales ojala y les sea util' + end
			break
		# ---------------------------------------------------------------------------------
		else:
			print red + 'mala desicion!' + end
		# ---------------------------------------------------------------------------------
		while raw_input('ingresa {0}M{1} para regresar al menu anterrior: '.format(red, end)).lower() != 'm':
			print red + 'Mala desicion!' + end


