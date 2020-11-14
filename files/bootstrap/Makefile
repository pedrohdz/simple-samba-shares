NOBODY_UID ?= 65534
NOBODY_GID ?= 65534

build_dir := /opt/bootstrap/build
etc_src_dir := /etc
etc_trg_files := $(addprefix $(build_dir)/, group passwd shadow)
runtime_dir := /opt/samba/runtime
smb_conf_file := /opt/samba/conf/smb.conf
smb_conf_include_dir := /opt/samba/conf/include
smb_conf_src_dir := /opt/bootstrap
smb_conf_src_file := $(smb_conf_src_dir)/smb.conf
user_file := /opt/bootstrap/secrets/users.conf

share_dir := /share
public_share_dir := $(share_dir)/public
share_dirs := $(public_share_dir)

root_dirs := \
	/opt/bootstrap/build \
	/opt/samba \
	/opt/samba/conf \
	/opt/samba/conf/include \
	/opt/samba/runtime \
	/share

runtime_dirs := $(addprefix $(runtime_dir)/, cache lock logs ncalrpc ntp pid private state)
smb_conf_include_files := $(addprefix $(smb_conf_include_dir)/, global.conf shares.conf)

user_sentinels := $(foreach _user, \
	$(shell grep -v '\s*\#' $(user_file) | cut -d: -f1), \
	$(build_dir)/$(_user)-samba-account-created)


.PHONY: all change-nobody prepare save-etc-files users

all: prepare save-etc-files

#----
# Saving the /etc/ files
#----
save-etc-files: $(etc_trg_files)

$(etc_trg_files): | $(root_dirs)

$(build_dir)/%: $(etc_src_dir)/% $(user_sentinels)
	cp -vp '$<' '$@'


#----
# Creating users
#----
users: $(user_sentinels)

$(user_sentinels): | prepare

$(build_dir)/%-samba-account-created: $(build_dir)/%-system-account-created
	@echo '+ Creating Samba user "$*" and setting password'
	set -eu ; \
		PASSWORD=$$(grep '^$*:' '$(user_file)' | cut -d: -f2-) ;\
		if [ -z "$$PASSWORD" ]; then echo '+ ERROR - Empty password not allowed!' ; exit 1; fi ;\
		echo -e "$$PASSWORD\n$$PASSWORD" | smbpasswd -c $(smb_conf_file) -s -a '$*'
	find $(runtime_dir)/* ! \( -user nobody -a -group nobody \) -exec chown nobody:nobody -vR '{}' \;
	touch '$@'

$(build_dir)/%-system-account-created:
	@echo '+ Creating system user "$*"'
	adduser -D -H -h '/nowhere' -s '/sbin/nologin' -G 'nogroup' -g 'Samba user' '$*'
	touch '$@'


#----
# Preparing the filesystem
#----
prepare: | $(runtime_dirs) $(share_dirs) $(smb_conf_file) \
	$(smb_conf_include_files) $(public_share_dir)/README.txt /var/log/samba

$(root_dirs): %:
	install -d '$@'

$(smb_conf_file): $(smb_conf_src_file) | $(root_dirs)
	install -m 0444 '$<' '$@'

$(smb_conf_include_dir)/%: $(smb_conf_src_dir)/% | $(root_dirs)
	install -m 0444 '$<' '$@'

$(runtime_dirs): %: | $(root_dirs)
	install -o nobody -g nobody -d '$@'

$(share_dirs): %: | $(root_dirs)
	install -m 2775 -o nobody -g nobody -d '$@'

$(public_share_dir)/README.txt: $(smb_conf_src_dir)/test-share-readme.txt
	install -m 0444 -o root -o root '$<' '$@'

# Have not found a way to move /var/log/samba/cores to runtime_dir.  Using a
# symlink for now.
/var/log/samba: $(runtime_dir)/logs
	rmdir '$@'
	ln -s '$<' '$@'


#----
# Changing nobody UID and GID
#----
NOBODY_OLD_UID := $(shell id -u nobody)
NOBODY_OLD_GID := $(shell id -g nobody)

change-nobody:
	@echo '+ Ensure nobody has a UID of $(NOBODY_UID)'
	set -eu ; \
		if [ $(NOBODY_OLD_UID) -ne $(NOBODY_UID) ]; then \
			find $(runtime_dir) -user nobody -exec chown -v $(NOBODY_UID) '{}' \; ;\
			usermod --uid $(NOBODY_UID) nobody ;\
		fi
	@echo '+ Ensure nobody has a GID of $(NOBODY_GID)'
	set -eu ; \
		if [ $(NOBODY_OLD_GID) -ne $(NOBODY_GID) ]; then \
			find $(runtime_dir) -group nobody -exec chgrp -v $(NOBODY_GID) '{}' \; ;\
			groupmod --gid $(NOBODY_GID) nobody ;\
		fi

