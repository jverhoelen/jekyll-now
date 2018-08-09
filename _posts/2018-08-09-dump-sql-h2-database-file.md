---
layout: post
title: "How to dump SQL from a H2 database file"
category: "Databases"
---

Recently I ran into the situation that I had to migrate a file basd H2 database to another relational database. At the beginning I was kind of scared that there will be no proper and quick solution. After a bit of research I found a very easy way that I would like to share with you. We also need to have Java installed and get a little utility from H2.

## Download H2, prepare script and... Fire!

Let's `cd` into the directory where our `.mv.db` file is stored, let's call it `vocablesDb.mv.db` for now. Then we will download H2 as JAR-file:

```bash
wget -O h2.jar http://repo2.maven.org/maven2/com/h2database/h2/1.4.197/h2-1.4.197.jar
```

Now we will create the file `query.sql`. It contains a simple SQL instruction to dump the databases schema and data to a file called `db-dump.sql`:

```bash
echo "SCRIPT TO 'db-dump.sql'" > query.sql
```

Now we can connect the dots and use a utility from the H2 Java library:

```bash
java -cp h2.jar org.h2.tools.RunScript -url "jdbc:h2:file:./vocablesDb" -user username -password pazzword -script query.sql -showResults
```

This should be pretty much everything. You will find the complete SQL dump in `db-dump.sql` now. If you get an error, H2 will tell you pretty much what you did wrong. Maybe you got the URL or credentials wrong?!