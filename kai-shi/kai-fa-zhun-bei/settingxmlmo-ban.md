# setting.xml模板

校验日期：

---

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings>
	<localRepository>{你的仓库地址}</localRepository>
	<servers>
		<server>
			<id>user-release</id>
			<username>deployment</username>
			<password><![CDATA[deployment@belle]]></password>
		</server>
		<server>
			<id>user-snapshot</id>
			<username>deployment</username>
			<password><![CDATA[deployment@belle]]></password>
		</server>
		<server>
			<id>user-releases</id>
			<username>deployment</username>
			<password><![CDATA[deployment@belle]]></password>
		</server>
		<server>
			<id>user-snapshots</id>
			<username>deployment</username>
			<password><![CDATA[deployment@belle]]></password>
		</server>
	</servers>
	<mirrors>
		<mirror>
			<!--This sends everything else to /public -->
			<id>nexus</id>
			<mirrorOf>*</mirrorOf>
			<url>http://nexus.belle.cn:8081/nexus/content/groups/public</url>
		</mirror>
	</mirrors>
	<proxies />
	<profiles>
		<profile>
			<id>nexus</id>
			<repositories>
				<repository>
					<id>nexus</id>
					<name>local private nexus</name>
					<url>http://nexus.belle.cn:8081/nexus/content/groups/public</url>
				</repository>
			</repositories>
		</profile>
		<profile>
			<id>nexus-snapshots</id>
			<repositories>
				<repository>
					<id>nexus-snapshots</id>
					<name>local private nexus snapshots</name>
					<url>http://192.168.1.91:8081/nexus/content/groups/public-snapshots</url>
				</repository>
			</repositories>
		</profile>
	</profiles>

	<activeProfiles>
		<activeProfile>nexus</activeProfile>
		<activeProfile>nexus-snapshots</activeProfile>
	</activeProfiles>
</settings>

```

> 修改模板中的仓库地址。



