# Adds functions missing in stdlib

# Bit of a hack
echo = find_program('echo')
placeholder_src = custom_target('placeholder_src',
                        output : 'placeholder.c',
                        capture: true,
                        command : [echo, '/* placeholder */'])

if host_system == 'linux'
	misc_lib = static_library('misc', ['strlcpy.c', 'strlcat.c'],
	include_directories: [inc],
	install : false)
elif host_system == 'darwin'
	# placeholder
	misc_lib = static_library('misc', placeholder_src)
elif host_system == 'windows'
	misc_lib = static_library('misc', ['strcasestr.c', 'strlcat.c', 'strlcpy.c', 'strsep.c', 'strtok_r.c'],
		include_directories: [inc],
		install: false)
else
	misc_lib = static_library('misc', empty_src)
	message('@0@ build is not tested'.format(host_system))
endif
