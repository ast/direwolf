# Adds functions missing in stdlib

if host_system == 'linux'
	misc_lib = static_library('misc', ['strlcpy.c', 'strlcat.c'],
	include_directories: [inc],
	install : false)
elif host_system == 'darwin'
	# placeholder
	misc_lib = static_library('misc')
elif host_system == 'windows'
	misc_lib = static_library('misc', ['strcasestr.c', 'strlcat.c', 'strlcpy.c', 'strsep.c', 'strtok_r.c'],
		include_directories: [inc],
		install: false)
else
	misc_lib = static_library('misc')
	message('untested system')
endif
