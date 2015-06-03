# Digital Preservation the Hard Way
###Open Repositories 2015 Poster Session
###June 9, 2015

##Helpful SQL

*NOTE:* all of the following SQL assumes you are looking for objects related
to a deleted community. If you are borrowing this SQL, be sure to replace
DELETED-COMM-ID with a value meaningful to you.

*WARNING:* this SQL was developed for an Oracle database. It seems like
pretty standard SQL, but what do I know, I'm an Oracle user. If you are
not, proceed with caution.

###SQL for epersongroup (needed to head off foreign key violations, add this first)
```sql
SELECT * from epersongroup where eperson_group_id in ( select submitter
from collection where COLLECTION_ID in (SELECT collection_id FROM
community2collection WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID))
```

###SQL for missing communities:
```sql
SELECT * FROM community WHERE community_id IN (SELECT CHILD_COMM_ID
FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID
```

###SQL for the relationships for missing communities:
```sql
SELECT * FROM community2community WHERE child_comm_id IN (SELECT
CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = 
DELETED-COMM-ID) or child_comm_id = DELETED-COMM-ID
```

###SQL for missing collections:
```sql
SELECT * from collection where COLLECTION_ID in (SELECT collection_id
FROM community2collection WHERE community_id IN (SELECT CHILD_COMM_ID
FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID)
```

###SQL for the relationships for missing collections:
```sql
SELECT * FROM community2collection WHERE community_id IN (SELECT
CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = 
DELETED-COMM-ID) or community_id=DELETED-COMM-ID
```

###SQL for missing items:
```sql
select * from ITEM where OWNING_COLLECTION in (SELECT collection_id
FROM community2collection WHERE community_id IN (SELECT CHILD_COMM_ID
FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID)
```

###SQL for missing bundles:
```sql
SELECT * from BUNDLE WHERE BUNDLE_ID IN (select BUNDLE_ID from
ITEM2BUNDLE WHERE ITEM_ID IN ( select ITEM_ID from ITEM where
OWNING_COLLECTION in (SELECT collection_id FROM community2collection
WHERE community_id IN (SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY
WHERE PARENT_COMM_ID = DELETED-COMM-ID) or community_id = 
DELETED-COMM-ID) ) )
```

###SQL for missing bitstreams:
```sql
SELECT * from BITSTREAM WHERE BITSTREAM_ID IN (SELECT BITSTREAM_ID FROM
BUNDLE2BITSTREAM WHERE BUNDLE_ID IN (SELECT BUNDLE_ID from BUNDLE WHERE
BUNDLE_ID IN (select BUNDLE_ID from ITEM2BUNDLE WHERE ITEM_ID IN (
select ITEM_ID from ITEM where OWNING_COLLECTION in (SELECT
collection_id FROM community2collection WHERE community_id IN (SELECT
CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = 
DELETED-COMM-ID) or community_id = DELETED-COMM-ID)))))
```

###SQL for missing handles for items:
```sql
SELECT * FROM handle WHERE resource_id in (select ITEM_ID from ITEM
where OWNING_COLLECTION in (SELECT collection_id FROM
community2collection WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID)) AND resource_type_id=2
```

###SQL for missing handles for collections:
```sql
SELECT * FROM handle WHERE resource_id in (SELECT COLLECTION_ID from
collection where COLLECTION_ID in (SELECT collection_id FROM
community2collection WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID)) AND resource_type_id=3
```

###SQL for missing handles for communities:
```sql
SELECT * FROM handle WHERE resource_id in ( SELECT COMMUNITY_ID FROM
community WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID ) AND resource_type_id=4 
```

###SQL for METADATAVALUE table (item metadata)
```sql
SELECT metadata_value_id,item_id,metadata_field_id,replace(text_value,
'''','''''') as TEXT_VALUE,text_lang,place,authority,confidence FROM
RESTORED-SNAPSHOT-DB.metadatavalue WHERE item_id IN (select ITEM_ID
from RESTORED-SNAPSHOT-DB.ITEM where OWNING_COLLECTION in (SELECT
collection_id FROM RESTORED-SNAPSHOT-DB.community2collection WHERE
community_id IN (SELECT CHILD_COMM_ID FROM RESTORED-SNAPSHOT-DB.
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID));
```

###SQL for ITEM2BUNDLE relationships
```sql
SELECT * FROM item2bundle WHERE item_id IN (select ITEM_ID from ITEM
where OWNING_COLLECTION in (SELECT collection_id FROM
community2collection WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID))
```

###SQL for BUNDLE2BITSTREAM relationships
```sql
SELECT * FROM bundle2bitstream WHERE bundle_id IN (select BUNDLE_ID
from BUNDLE WHERE BUNDLE_ID IN (select BUNDLE_ID from ITEM2BUNDLE WHERE
ITEM_ID IN ( select ITEM_ID from ITEM where OWNING_COLLECTION in
(SELECT collection_id FROM community2collection WHERE community_id IN
(SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID =
DELETED-COMM-ID) or community_id = DELETED-COMM-ID) ) ))
```

###SQL for COLLECTION2ITEM relationships
```sql
SELECT * FROM collection2item WHERE collection_id IN (SELECT
collection_id FROM community2collection WHERE community_id IN (SELECT
CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = 
DELETED-COMM-ID) or community_id = DELETED-COMM-ID)
```

###SQL for RESOURCEPOLICY
```sql
SELECT * FROM resourcepolicy WHERE (resource_type_id=0 AND resource_id
IN ( SELECT BITSTREAM_ID FROM BUNDLE2BITSTREAM WHERE BUNDLE_ID IN
(SELECT BUNDLE_ID from BUNDLE WHERE BUNDLE_ID IN (select BUNDLE_ID from
ITEM2BUNDLE WHERE ITEM_ID IN ( select ITEM_ID from ITEM where
OWNING_COLLECTION in (SELECT collection_id FROM community2collection
WHERE community_id IN (SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY
WHERE PARENT_COMM_ID = DELETED-COMM-ID) or community_id = 
DELETED-COMM-ID)))) )) OR (resource_type_id=1 AND resource_id IN (
SELECT BUNDLE_ID from ITEM2BUNDLE WHERE ITEM_ID IN ( select ITEM_ID
from ITEM where OWNING_COLLECTION in (SELECT collection_id FROM
community2collection WHERE community_id IN (SELECT CHILD_COMM_ID FROM
COMMUNITY2COMMUNITY WHERE PARENT_COMM_ID = DELETED-COMM-ID) or
community_id = DELETED-COMM-ID) ) )) OR (resource_type_id=2 AND
resource_id IN ( SELECT ITEM_ID from ITEM where OWNING_COLLECTION in
(SELECT collection_id FROM community2collection WHERE community_id IN
(SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE 
PARENT_COMM_ID = DELETED-COMM-ID) or community_id = 
DELETED-COMM-ID) ) ) OR (resource_type_id=3 AND resource_id IN 
( SELECT collection_id FROM community2collection WHERE community_id IN
(SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE 
PARENT_COMM_ID = DELETED-COMM-ID) or community_id = 
DELETED-COMM-ID )) OR (resource_type_id=4 AND (resource_id IN 
( SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE 
PARENT_COMM_ID = DELETED-COMM-ID )) OR resource_id = DELETED-COMM-ID) 
```

###SQL for producing the update query for community handles
```sql
SELECT 'UPDATE HANDLE SET HANDLE=''' || HANDLE || ''',
RESOURCE_TYPE_ID=4, RESOURCE_ID=' || RESOURCE_ID || ' WHERE 
HANDLE_ID=' || HANDLE_ID || ';' AS "REM UPDATING COMMUNITY HANDLES"
FROM handle WHERE resource_id in ( SELECT COMMUNITY_ID FROM community
WHERE community_id IN (SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY
WHERE PARENT_COMM_ID = DELETED-COMM-ID) or community_id = DELETED-COMM-ID ) AND resource_type_id=4
```

###SQL for producing the update query for collection handles
```sql
SELECT 'UPDATE HANDLE SET HANDLE=''' || HANDLE || ''',
RESOURCE_TYPE_ID=3, RESOURCE_ID=' || RESOURCE_ID || ' WHERE 
HANDLE_ID=' || HANDLE_ID || ';' AS "REM UPDATING COLL HANDLES" FROM
handle WHERE resource_id in (SELECT COLLECTION_ID from collection where
COLLECTION_ID in (SELECT collection_id FROM community2collection WHERE
community_id IN (SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY WHERE
PARENT_COMM_ID = DELETED-COMM-ID) or community_id = DELETED-COMM-ID))
AND resource_type_id=3
```

###SQL for producing the update query for item handles
```sql
SELECT 'UPDATE HANDLE SET HANDLE=''' || HANDLE || ''',
RESOURCE_TYPE_ID=2, RESOURCE_ID=' || RESOURCE_ID || ' WHERE 
HANDLE_ID=' || HANDLE_ID || ';' AS "REM UPDATING ITEM HANDLES" FROM
handle WHERE resource_id in (select ITEM_ID from ITEM where
OWNING_COLLECTION in (SELECT collection_id FROM community2collection
WHERE community_id IN (SELECT CHILD_COMM_ID FROM COMMUNITY2COMMUNITY
WHERE PARENT_COMM_ID = DELETED-COMM-ID) or community_id = 
DELETED-COMM-ID)) AND resource_type_id=2
```
