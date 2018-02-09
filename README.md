# Important
[pylibaio]:put the folder in /usr/local/lib/python2.7/ and rename to libaio/ <p>
Create folder to save mappings:/tmp/mappings/ <p>
Default Log location:/etc/config/qsnapsync/logs <p>
Default Log format:snapsync_err.log<p>
Additional Log config:snapsync.cfg in /etc/config/snapsync.cfg, format like below <p>

## Snapsync.cfg format
[Logging]
directory=/tmp
file_level=10

## [Snapsync Usage]
3 roles for snapsync usage:
	[master] for remote,revert and local sync
	[qcow2_exporter] for local qcow2 backup
	[qcow2_importer] import from a qcow2 file

Parameters and Usage for 3 roles:
### [master]:
Parameters required:
	1. src: source snapshot path. ex. /dev/mapper/vg1-snap30001
	2. dst: destination device. ex. admin@192.168.80.88:/dev/mapper/thin
	3. meta: metadata device of pool which synced thin/thick device resides on
	4. id: device ID of synced thin/thick device by cmd `lvs -o+thin_id`
	5. uuid: uuid used to identify the source lv, recommand to use lvm uuid
Parameters optional:
	1. port: ssh port
	2. timeout: disconnect from backup destination if this timeout time has expired
	3. speed-limit: transfer speed limit, support suffix G, M, B
	4. encrypt: enable aes-256-cbc encryption algorithm to protect blocks during transfer
	5. resume: resume transmission from this byte offset
Usage for three sub-roles:
	1. Remote sync (encrypt enabled):
		/usr/local/bin/python snapsync.py master --port 22 --meta /dev/mapper/vg1-tp1_tmeta --id 4 --src /dev/mapper/vg1-snap30001 --dst admin@192.168.80.88:/dev/mapper/thin --uuid 1 --encrypt
	2. Local sync (encrypt enabled):
		/usr/local/bin/python snapsync.py master --port 22 --meta /dev/mapper/vg1-tp1_tmeta --id 4 --src=/dev/mapper/vg1-snap30001 --dst /dev/mapper/thin --uuid 1 --encrypt
	3. Revert sync:
		/usr/local/bin/python revert_snapsync.py master --port 22 --meta /dev/mapper/vg1-tp1_tmeta --id 4 --src admin@192.168.80.88:/dev/mapper/vg1-snap30001 --dst admin@192.168.80.36:/dev/mapper/thin --uuid 1

### [qcow2_exporter]:
Parameters required:
	1. src: source device path. ex. /dev/mapper/vg1-snap30001
	2. qcow: destination QCOW2 path. ex. /mnt/HDA_ROOT/testqcow2
	3. size: source device size (in bytes)
	4. meta: metadata device of pool which synced thin/thick device resides on
	5. id: device ID of synced thin/thick device by cmd `lvs -o+thin_id`
Parameters optional:
	1. uuid: uuid used to identify the source lv, recommand to use lvm uuid
Usage:
	1. Local sync:
		/usr/local/bin/python snapsync.py qcow2_exporter --meta /dev/mapper/vg1-tp1_tmeta --id 4 --src /dev/mapper/vg1-snap30001 --uuid 1 --qcow /mnt/HDA_ROOT/testqcow2 --size 5368709120

### [qcow2_importer]:
Parameters required:
	1. src: source device path. ex. /mnt/HDA_ROOT/testqcow2
	2. dst: destination device. ex. /dev/mapper/thin
Parameters optional:
	1. uuid: uuid used to identify the source lv, recommand to use lvm uuid
Usage:
	1. Local sync:
		/usr/local/bin/python snapsync.py qcow2_importer --dst /dev/mapper/thin --uuid 1 --src /mnt/HDA_ROOT/testqcow2

