project('direwolf', 'c',
	version: 'dev',
	default_options : [ 'warning_level=3',
						'c_std=c11',
                      	'buildtype=debugoptimized'])

## How to generate configuration file
# Linux
# egrep '^C|^L' generic.conf | cut -c2-999 > direwolf.conf
# Mac
# egrep '^C|^M' generic.conf | cut -c2-999 > direwolf.conf

cc = meson.get_compiler('c')

add_project_arguments(['-D_XOPEN_SOURCE=600', '-D_DEFAULT_SOURCE=1'], language: 'c')

## Raspberry Pi optimzations
# add_project_arguments(['-O3', '-ffast-math', '-mfpu=neon-vfpv4', '-funsafe-math-optimizations'], language: 'c')

host_system = host_machine.system()

if host_system == 'linux'
	# Use ALSA
	audio_dep = dependency('asound', required: true)
	add_project_arguments('-DUSE_ALSA', language: 'c')
	audio_c = 'audio.c'
elif host_system == 'darwin'
	# Use portaudio
	audio_dep = dependency('portaudio-2.0', required: true)
	add_project_arguments('-DUSE_PORTAUDIO', language: 'c')
	audio_c = 'audio_portaudio.c'
elif host_system == 'windows'
	# Use windows audio
	# need to add correct dependency here probably
	audio_dep = dependency('', required: true)
	audio_c = 'audio_win.c'
	message('windows build is not tested')
else
	# try OSS
	audio_dep = dependency('', required: true)
	audio_c = 'audio.c'
	message('@0@ build is not tested'.format(host_system))
endif

## Required
m_dep = cc.find_library('m', required : true)
thread_dep = dependency('threads', required: true)
## Optional
hamlib_dep = dependency('hamlib', required: false)
libgps_dep = dependency('libgps', required: false)

# TUDO: add libudev support

if hamlib_dep.found()
	add_project_arguments('-DUSE_HAMLIB', language: 'c')
endif

if libgps_dep.found()
	add_project_arguments('-DENABLE_GPSD', language: 'c')
endif

## Geotranz
subdir('geotranz')

## Misc
# subdir('misc')

## Include directories
inc = include_directories(['geotranz'])

textcolor_lib = static_library('textcolor',
    'textcolor.c',
    install: false)

# Generates fsk_fast_filter.h
gen_fff = executable('gen_fff',
	['demod_afsk.c', 'dsp.c'],
	link_with: textcolor_lib,
	c_args: ['-DGEN_FFF'],
	install: false
)

fsk_fast_filter_h = custom_target('fsk_fast_filter_h',
	output : 'fsk_fast_filter.h',
	command : gen_fff,
	capture: true, # meson captures output to stdout and writes to file
	install: false)

## Utilities

## decode_aprs
decode_aprs_sources = ['decode_aprs.c', 'kiss_frame.c', 'ax25_pad.c', 'dwgpsnmea.c', 'dwgps.c', 'dwgpsd.c',
	'serial_port.c', 'symbols.c', 'fcs_calc.c', 'latlong.c', 'log.c',
	'telemetry.c', 'tt_text.c']
executable('decode_aprs',
	decode_aprs_sources,
	link_with: textcolor_lib,
	dependencies: [libgps_dep],
	install: false,
	c_args: ['-DDECAMAIN'])

## text2tt
executable('text2tt', 'tt_text.c',
	install: false,
	c_args: ['-DENC_MAIN'])

## tt2text
executable('tt2text', 'tt_text.c',
	install: false,
	c_args: ['-DDEC_MAIN'])

## ll2utm
executable('ll2utm', 'll2utm.c',
	link_with: [geotranz_lib, textcolor_lib],
	include_directories: [inc],
	install: false)

## utm2ll
executable('utm2ll', 'utm2ll.c',
	link_with: [geotranz_lib, textcolor_lib],
	include_directories: [inc],
	install: false)

## log2gpx
executable('log2gpx', 'log2gpx.c',
	link_with: textcolor_lib,
	install: false)

## gen_packets
gen_packets_sources = ['gen_packets.c', 'ax25_pad.c', 'hdlc_send.c', 'fcs_calc.c',
	'gen_tone.c', 'morse.c', 'dtmf.c', 'dsp.c']
executable('gen_packets', gen_packets_sources,
	link_with: textcolor_lib,
	install: false)

# Unit test for AFSK demodulator
atest_sources = ['atest.c', 'demod.c', 'demod_afsk.c', 'demod_psk.c', 'demod_9600.c',
	'dsp.c', 'hdlc_rec.c', 'hdlc_rec2.c', 'multi_modem.c', 'rrbb.c', 'fcs_calc.c', 'ax25_pad.c',
	'decode_aprs.c', 'dwgpsnmea.c', 'dwgps.c', 'dwgpsd.c', 'serial_port.c', 'telemetry.c',
	'dtime_now.c', 'latlong.c', 'symbols.c', 'tt_text.c']
executable('atest', atest_sources,
	link_with: textcolor_lib,
	dependencies: [libgps_dep],
	install: false)

# Multiple AGWPE network or serial port clients to test TNCs side by side
executable('aclients', ['aclients.c', 'ax25_pad.c', 'fcs_calc.c'],
	link_with: textcolor_lib,
	install: false)

# Talk to a KISS TNC
executable('kissutil', ['kissutil.c', 'kiss_frame.c', 'ax25_pad.c', 'fcs_calc.c', 'serial_port.c',
	'dtime_now.c', 'sock.c'],
	link_with: textcolor_lib,
	c_args: ['-DKISSUTIL'],
	install: false)

# List USB audio adapters than can use GPIO for PTT.
executable('cm108', 'cm108.c',
	link_with: textcolor_lib,
	c_args: ['-DCM108_MAIN'],
	install: false)

# Touch Tone to Speech sample application.
executable('ttcalc', ['ttcalc.c', 'ax25_pad.c', 'fcs_calc.c'],
	link_with: textcolor_lib,
	install: false)

## Tests
# Unit test for inner digipeater algorithm
dtest = executable('dtest', ['digipeater.c', 'dedupe.c', 'pfilter.c', 'ax25_pad.c',
	'fcs_calc.c', 'tq.c', 'decode_aprs.c', 'dwgpsnmea.c', 'dwgps.c', 'dwgpsd.c',
	'serial_port.c', 'latlong.c', 'telemetry.c', 'symbols.c', 'tt_text.c'],
	link_with: textcolor_lib,
	dependencies: [libgps_dep],
	c_args: ['-DDIGITEST'],
	install: false)
test('dtest', dtest)

# Unit test for APRStt tone sequence parsing.
ttest = executable('ttest', ['aprs_tt.c', 'tt_text.c', 'latlong.c'],
	link_with: [geotranz_lib, textcolor_lib],
	include_directories: [inc],
	c_args: ['-DTT_MAIN'],
	install: false)
test('ttest', ttest)

# Unit test for APRStt tone sequence / text conversions.
tttexttest = executable('tttexttest', 'tt_text.c',
	link_with: textcolor_lib,
	# include_directories: [inc],
	c_args: ['-DTTT_TEST'],
	install: false)
test('tttexttest', tttexttest)

# Unit test for Packet Filtering.
pftest = executable('pftest', ['pfilter.c', 'ax25_pad.c', 'fcs_calc.c', 'decode_aprs.c', 'dwgpsnmea.c', 'dwgps.c',
	'dwgpsd.c', 'serial_port.c', 'latlong.c', 'symbols.c', 'telemetry.c', 'tt_text.c'],
	link_with: textcolor_lib,
	dependencies: libgps_dep,
	c_args: ['-DPFTEST'],
	install: false)
test('pftest', pftest)

# Unit test for telemetry decoding.
tlmtest = executable('tlmtest', ['telemetry.c', 'ax25_pad.c', 'fcs_calc.c'],
	link_with: textcolor_lib,
	c_args: ['-DTEST'],
	install: false)
test('tlmtest', tlmtest)

# Unit test for location coordinate conversion.
lltest = executable('lltest', 'latlong.c',
	link_with: textcolor_lib,
	c_args: ['-DLLTEST'],
	install: false)
test('lltest', lltest)

# Unit test for encoding position & object report.
enctest = executable('enctest', ['encode_aprs.c', 'latlong.c'],
	link_with: textcolor_lib,
	c_args: ['-DEN_MAIN'],
	install: false)
test('enctest', enctest)

# Unit test for KISS encapsulation.
kisstest = executable('kisstest', 'kiss_frame.c',
	c_args: ['-DKISSTEST'],
	install: false)
test('kisstest', kisstest)

# Unit test for constructing frames besides UI.
pad2test = executable('pad2test', ['ax25_pad2.c', 'ax25_pad.c', 'fcs_calc.c'],
	link_with: textcolor_lib,
	c_args: ['-DPAD2TEST'],
	install: false)
test('pad2test', pad2test)

# Unit Test for XID frame encode/decode.
xidtest = executable('xidtest', 'xid.c',
	link_with: textcolor_lib,
	c_args: ['-DXIDTEST'],
	install: false)
test('xidtest', xidtest)

# Unit Test for DTMF encode/decode.
dtmftest = executable('dtmftest', 'dtmf.c',
	link_with: textcolor_lib,
	c_args: ['-DDTMF_TEST'],
	install: false)
test('dtmftest', dtmftest)

## Direwolf main application
direwolf_sources = ['direwolf.c', 'config.c', 'recv.c', 'demod.c', 'dsp.c', 'demod_afsk.c', 'demod_psk.c',
		'demod_9600.c', 'hdlc_rec.c', 'hdlc_rec2.c', 'multi_modem.c', 'rdq.c', 'rrbb.c', 'dlq.c',
		'fcs_calc.c', 'ax25_pad.c', 'ax25_pad2.c', 'xid.c', 'decode_aprs.c', 'symbols.c', 'server.c',
		'kiss.c', 'kissserial.c', 'kissnet.c', 'kiss_frame.c', 'hdlc_send.c', 'fcs_calc.c',
		'gen_tone.c', 'audio_stats.c', 'digipeater.c', 'cdigipeater.c', 'pfilter.c',
		'dedupe.c', 'tq.c', 'xmit.c', 'morse.c', 'ptt.c', 'beacon.c', 'encode_aprs.c', 'latlong.c',
		'encode_aprs.c', 'latlong.c', 'dtmf.c', 'aprs_tt.c', 'tt_user.c', 'tt_text.c', 'igate.c',
		'waypoint.c', 'serial_port.c', 'log.c', 'telemetry.c', 'dwgps.c', 'dwgpsnmea.c', 'dwgpsd.c',
		'dtime_now.c', 'mheard.c', 'ax25_link.c', 'cm108.c', audio_c, fsk_fast_filter_h]

executable('direwolf',
	direwolf_sources,
	link_with: [geotranz_lib, textcolor_lib],
	dependencies: [audio_dep, hamlib_dep, libgps_dep, thread_dep, m_dep],
	include_directories: [inc],
	install: true)
