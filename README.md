# PowerSchool_Plugin_Template

Template repo for creating PowerSchool Plugins

This template provides:

* Basic structure for plugins
* Bash script for packaging and versioning plugins
* `BAT` script for automating uploads from PowerSchool server
* `README` template

## Using a Template

1. On GitHub.com navigate to the main page of this repo (probably where you are)
2. Choose *Use this Template*
![Use this Template](https://docs.github.com/assets/cb-77734/mw-1440/images/help/repository/use-this-template-button.webp)
3. Select *Create a new repository*
4. Name your repository
5. Clone your new repo
6. Remove this section of the `README.md`

# My Plugin Name Here

Plugin description

## Updating and Packaging

Following any updates to this plugin, it is necessary to do the following:

1. Bump the version number in `plugin.xml` -- PowerSchool will not allow a plugin to be updated unless the plugin version number is larger than the installed version number.
2. Zip the plugin in a flat archive. Use the `package.sh` script: `$ package.sh /Plugin_name` -- using the MacOS built in *Archive* tool will create a plugin .zip file that cannot be used.

## Usage Information

Add any relevant usage information

## Configuration Information

Any relevant information needed to configure the plugin such as Data Export Manager Settings.

## Plugin Documentation

Useful documentation for PowerSchool Plugins

### PowerSchool Documentation

A PowerSchool support login is needed to access this content.

#### [PowerSchool Plugin Developer Documentation](https://support.powerschool.com/developer/#/page/plugins)

* [Plugin XML](https://support.powerschool.com/developer/#/page/plugin-xml)
* [Plugin Package Structure](https://support.powerschool.com/developer/#/page/plugin-zip)
* [PowerQuery Structure](https://support.powerschool.com/developer/#/page/powerqueries)

### Other Useful Documentation

#### [PowerSchool Customization - Coding Basic HTML, CSS, LTK, PSHTML and Tools](https://www.psugcal.org/images/4/4d/Customization_2_-_Coding_Basics_in_PowerSchool.pdf)

See FRN (File Reference  Numbers) - useful for creating direct links to student/parent records in output

## General Plugin Troubleshooting

### Returned Column References

The `columns` section of the XML document refer to the columns that will be offered in the _Data Export Manager_ screens. The text of the column name is arbitrary and can be anything, but the `<column column="TABLE.FIELD">` portion **must** refer to a "core" table of PowerSchool. When in doubt, use `column="STUDENTS.ID"`

```XML
<columns>
    <column column="STUDENTS.ID">Student_Number</column>
    <column column="STUDENTS.ID">Enroll_Status</column>
    <column column="STUDENTS.ID">Family_Ident</column>
</columns>
```

### Returned Column Number and Names

The total number of columns returned by the query must match exactly in name and number to the number of `column` references in the XML document.

Example:

```SQL
<queries>
    <!--set name here (also applies to permissions_root-->
    <query name="com.foo.bar" coreTable="students" flattened="false">
        <!--add description here-->
        <description>foo example</description>
        <!--number of columns here must match number sql returns-->
        <columns>
            <column column="STUDENTS.ID">Name</column>
            <column column="STUDENTS.ID">Date</column>
            <column column="STUDENTS.ID">Food</column>
         </columns>
        <!--SQL query in format <![CDATA[QUERY]]>-->
        <sql>
            <![CDATA[
            select
                foo.name        as "Name"
                foo.date        as "Date"
                foo.food        as "Food"
            from foobartable as foo
            ]]>
        </sql>
    </query>
</queries>
```

### Core Table

The `coreTable=` value must refer to a known PowerSchool core table. This determines which menu the named query appears in on the _Data Export Manager_ screen. When in doubt, use `students`

Example:

```XML
    <query name="com.foo.bar" coreTable="students" flattened="false">
```

### Things to Avoid

#### Excessively Long Query Names

Long query names will result in odd errors and an inability to install and run the Named Query. The limit is around 100 characters total

Example:

```xml
<query name="com.foo.spam_ham_ham_eggs_and_ham" coreTable="students" flattened="false">
```

Alternative:

```xml
<query name="com.foo.spam_ham_2" coreTable="students" flattened="false">
```

#### Wild-Card Column references in SELECT

While this is completely valid SQL, PowerQuery can't handle it and direct column references should be used. **NB!** There is a counter example to this where a wildcard must be used when using CTEs. See the errors section below for more information.

Example:

```SQL
Select *

From table table
```

When installing the plugin this will result in an error that indicates that the query cannot determine a column name

Alternative:

```SQL
-- select the first column
Select 1

From table table
```

### Errors

When enabling a plugin it will be validated and sometimes kick errors associated with the format of the SQL. Some errors will also manifest when running a plugin from the *Data Export Manager* screens.

#### Plugin Install Message:  `cannot determine table name of column XXX`

**Case 1 - ORDER BY Mismatch**
This appears to be due to a column that is referenced by only the column name in an `ORDER BY` clause.

Example:

```SQL
ORDER BY foobar
```

Alternative:

```SQL
ORDER BY spam.foobar
```

**Case 2 - Using CTE Tables**

When using CTE joins, you may need to use a `SELECT *` rather than a table alias.

Example:

```SQL
    -- CTE use:
    LEFT JOIN sca_complete contact1 ON contact1.studentdcid = s.dcid AND contact1.contactprio = 1
    LEFT JOIN sca_complete contact2 ON contact2.studentdcid = s.dcid AND contact2.contactprio = 2
    WHERE s.enroll_status = 0
    ORDER BY s.lastfirst
```

Alternative:

```SQL
    LEFT JOIN (SELECT * FROM sca_complete) cone ON cone.studentdcid = s.dcid AND cone.contactprio = 1
    LEFT JOIN (SELECT * FROM sca_complete) ctwo ON ctwo.studentdcid = s.dcid AND ctwo.contactprio = 2
    WHERE s.enroll_status = 0
    ORDER BY s.lastfirst

```

#### Data Export Manger Errors

* `Unable to execute the query operation due to an invalid parameter. Update your filter values and try again.`
* `An unexpected error occurred while communicating with the server. Please contact your administrator`

These errors tend to be associated with using `order by` statements that are valid SQL, but do not refer to selected columns. To resolve this issue:

* Ensure that all `order by` statements in the SQL query are fields that are directly represented in the `select` section.
* Entirely remove the `order by` statements -- in some cases this resolves the above error entirely

Example:

```SQL
select distinct
    'enrollment' as "type",
    'UPDATE' as "action",
    'T_'||teachers.teachernumber as "child_code",
    'Instructor' as "role_name",
    teachers.homeschoolid as "parent_code"
 from TEACHERS TEACHERS
 where teachers.status =1
    and length(teachers.email_addr) >0
/*
note that the teacher number is not a directly select'd statement.
In this case it is concat'd to 'T_'
*/
 order by teachers.teachernumber asc
```

Alternative:

```SQL
select distinct
    'enrollment' as "type",
    'UPDATE' as "action",
    'T_'||teachers.teachernumber as "child_code",
    'Instructor' as "role_name",
    teachers.homeschoolid as "parent_code"
 from TEACHERS TEACHERS
 where teachers.status =1
    and length(teachers.email_addr) >0
/*
homeschoolid is directly select'd
*/
 order by teachers.homeschoolid asc
