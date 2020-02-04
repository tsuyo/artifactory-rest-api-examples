## Basic Steps

**1. Create a new admin user [You will be using it to do the rest of these commands]**
```
$ curl -u admin -X PUT --data @./01create_admin.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/security/users/tsuyo
```

**2. Create a new group**
```
$ curl -u tsuyo -X PUT --data @./02create_group.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/security/groups/tsuyo_group
```

**3. Update the group to contain the new admin user**
```
$ curl -u tsuyo -X POST --data @./03update_group.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/security/groups/tsuyo_group
```

**4. Create a new generic local repository**

For test
```
$ curl -u tsuyo -X PUT --data @./04create_repo.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/repositories/test-tsuyo-local
```
For production
```
$ curl -u tsuyo -X PUT --data @./04create_repo.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/repositories/prod-tsuyo-local
```

**5. Create a permission target containing the new group that grants all permissions to the new local repo.**
```
$ curl -u tsuyo -X PUT --data @./05create_permission.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/security/permissions/tsuyo_permission
```

**6. Deploy a file to the repo**
```
$ touch a.jar
# build.name & build.number are required to show a correct path (jar) in Builds on Artifactory UI
$ curl -u tsuyo -X PUT -T a.jar 'http://35.165.190.156:8081/artifactory/test-tsuyo-local/a.jar;build.name=tsuyo_build;build.number=1'
{
  "repo" : "test-tsuyo-local",
  "path" : "/a.jar",
  "created" : "2020-02-04T02:55:35.742Z",
  "createdBy" : "tsuyo",
  "downloadUri" : "http://35.165.190.156:8081/artifactory/test-tsuyo-local/a.jar",
  "mimeType" : "application/java-archive",
  "size" : "0",
  "checksums" : {
    "sha1" : "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "md5" : "d41d8cd98f00b204e9800998ecf8427e",
    "sha256" : "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  },
  "originalChecksums" : {
    "sha256" : "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  },
  "uri" : "http://35.165.190.156:8081/artifactory/test-tsuyo-local/a.jar"
}
```

**7. Deploy a build containing the file as the build module**
```
$ curl -u tsuyo -X PUT --data @./07deploy_build.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/build
```

**8. Promote the build to a production repo**
```
$ curl -u tsuyo -X POST --data @./08promote_build.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/build/promote/tsuyo_build/1
```

**9. Perform an AQL search that finds your file**
```
$ curl -u tsuyo -X POST --data @./09aql.txt -H "Content-Type: text/plain" http://35.165.190.156:8081/artifactory/api/search/aql
{
"results" : [ {
  "repo" : "prod-tsuyo-local",
  "path" : ".",
  "name" : "a.jar",
  "type" : "file",
  "size" : 0,
  "created" : "2020-02-04T02:55:35.742Z",
  "created_by" : "tsuyo",
  "modified" : "2020-02-04T02:55:35.742Z",
  "modified_by" : "tsuyo",
  "updated" : "2020-02-04T02:55:35.743Z"
} ],
"range" : {
  "start_pos" : 0,
  "end_pos" : 1,
  "total" : 1
}
}
```

**10. Delete the build and the file (Same command)**
```
$ curl -u tsuyo -X POST --data @./10delete_build.json -H "Content-Type: application/json" http://35.165.190.156:8081/artifactory/api/build/delete
The following builds have been deleted successfully: 'tsuyo_build#1'.
```

**11. Delete the generic local repository**
```
$ curl -u tsuyo -X DELETE http://35.165.190.156:8081/artifactory/api/repositories/test-tsuyo-local
$ curl -u tsuyo -X DELETE http://35.165.190.156:8081/artifactory/api/repositories/prod-tsuyo-local
```

**12. Delete the security target**
```
$ curl -u tsuyo -X DELETE http://35.165.190.156:8081/artifactory/api/security/permissions/tsuyo_permission
```

**13. Delete the group**
```
$ curl -u tsuyo -X DELETE http://35.165.190.156:8081/artifactory/api/security/groups/tsuyo_group
```

**14. Delete the user**
```
$ curl -u tsuyo -X DELETE http://35.165.190.156:8081/artifactory/api/security/users/tsuyo
```
