# update nextcloud read file
`*/5 * * * * docker exec nextcloud-app-1 php occ files:scan --all`

# change owner to smbuser
`*/5 * * * * chown -R smbuser:smbuser /hdrive/nextcloud_data`

# force https
```
<?php
$CONFIG = array (
  // other settings...
  'overwriteprotocol' => 'https',
);
```

# check user, then use it to config in the docker-compose.yml
`stat -c "%u:%g" /hdrive/nextcloud_data`

`sudo chmod -R 0770 /hdrive/nextcloud_data`