# jQGrid Integration Kit for Oracle PL/SQL and Application Express (Apex) #

## Introduction ##

I started developing applications back in the good (?) old client/server days. I was fortunate enough to discover [Delphi](http://en.wikipedia.org/wiki/Embarcadero_Delphi) quite early. Even from the start, the lowly 16-bit Delphi version 1 had a kick-ass [DBGrid control](http://delphi.about.com/od/usedbvcl/a/tdbgrid.htm) which allowed you to quickly and easily build data-centric applications. Just write a SQL statement in a TDataSet component, connect it to the grid, and voila! Instant multi-row display and editing out of the box, without any coding.

Fast forward a decade. While I do enjoy building web applications (with PL/SQL and Apex) these days, I've always missed the simplicity of that DBGrid in Delphi. Creating updateable grids with Apex is pretty tedious work (not being entirely satisfied with the built-in updateable tabular forms, I've employed a combination of the apex\_item API, page processes for updates and deletes, and custom-made Javascript helpers). It doesn't help that you have to refer to the tabular form arrays by number, rather than by name (g\_f01, g\_f02, etc.), and that you are restricted to a total of 50 columns per page.

Enter [jQGrid](http://www.trirand.com/blog/), "an Ajax-enabled JavaScript control that provides solutions for representing and manipulating tabular data on the web".

jQGrid can be integrated with any server-side technology, so I decided to integrate it with PL/SQL and Apex.

**The jQGrid Integration Kit for PL/SQL is free and open source.** Contact me if you want to contribute to the project.

<img src='http://jqgrid-for-plsql.googlecode.com/files/jqgrid_for_plsql_screenshot.jpg'>


<h2>Features</h2>

<ul><li>Single line of PL/SQL code to render grid<br>
</li><li>Populate data based on REF CURSOR or SQL text (with or without bind variables)<br>
</li><li>Define display modes (read only, sortable, editable) and edit types (checkbox, textarea, select list) per column<br>
</li><li>Store grid configuration in database, or specify settings via code (for read-only grids)<br>
</li><li>Ajax updates (insert, update, delete) based on either automatic row processing (dynamic SQL) or against your own package API<br>
</li><li>Multiple grids per page<br>
</li><li>Integrated logging and instrumentation<br>
</li><li>Usable without Apex (for stand-alone PL/SQL Web Toolkit applications) or with Apex, optionally integrated with Apex session security</li></ul>


<h2>Installation</h2>

Connect to your application schema (typically the schema that is associated with your Apex workspace) and run the <i>install.sql</i> script to create the database tables and PL/SQL packages.<br>
<br>
You must also copy the static files from the "js" and "img" folders to your web server.<br>
<br>
Check out the contents of the "demo" folder for examples of use.<br>
<br>
<br>
<h2>Configuration</h2>

The grid configuration is stored in a couple of (you guessed it) database tables. While it is certainly possible to edit the tables directly (see description of tables and columns below), it is probably easier to use the supplied API to load and save grid settings.<br>
<br>
The utility package JQGRID_UTIL_PKG contains a procedure, CREATE_COLUMN_DEFAULTS, that can save you from a lot of tedious configuration work, as long as your grid is based on a database table.<br>
<br>
<pre>

declare<br>
l_config jqgrid_pkg.t_grid_config;<br>
begin<br>
<br>
-- configure grid<br>
l_config.grid.grid_name := 'my_product_info_grid';<br>
l_config.grid.grid_title := 'Product Information';<br>
l_config.grid.grid_height := 200;<br>
l_config.grid.table_name := 'DEMO_PRODUCT_INFO'; -- from the Apex demo application<br>
l_config.grid.pk_column1 := 'PRODUCT_ID';<br>
l_config.grid.read_only := jqgrid_pkg.g_false;<br>
<br>
-- save configuration<br>
jqgrid_pkg.save_grid_config (l_config);<br>
<br>
-- add all columns from specified table, with appropriate defaults<br>
jqgrid_util_pkg.create_column_defaults (l_config, p_delete_existing_columns => true);<br>
<br>
commit;<br>
<br>
end;<br>
<br>
</pre>

Alternatively, or if your grid is based on a query that joins several tables, you define each column separately:<br>
<br>
<pre>

declare<br>
l_config jqgrid_pkg.t_grid_config;<br>
begin<br>
<br>
-- configure grid<br>
l_config.grid.grid_name := 'my_product_info_grid';<br>
l_config.grid.grid_title := 'Product Information';<br>
l_config.grid.grid_height := 200;<br>
l_config.grid.table_name := 'DEMO_PRODUCT_INFO'; -- from the Apex demo application<br>
l_config.grid.pk_column1 := 'PRODUCT_ID';<br>
l_config.grid.read_only := jqgrid_pkg.g_false;<br>
<br>
l_config.grid_columns(1).column_name := 'PRODUCT_NAME';<br>
l_config.grid_columns(1).display_name := 'Product Name';<br>
l_config.grid_columns(1).editable := jqgrid_pkg.g_true;<br>
<br>
l_config.grid_columns(2).column_name := 'PRODUCT_DESCRIPTION';<br>
l_config.grid_columns(2).display_name := 'Description';<br>
l_config.grid_columns(2).editable := jqgrid_pkg.g_true;<br>
l_config.grid_columns(2).edit_type := jqgrid_pkg.g_edit_type_text_area;<br>
<br>
l_config.grid_columns(3).column_name := 'LIST PRICE';<br>
l_config.grid_columns(3).display_name := 'Price ($)';<br>
l_config.grid_columns(3).editable := jqgrid_pkg.g_true;<br>
l_config.grid_columns(3).data_type := jqgrid_pkg.g_data_type_number;<br>
<br>
-- and so on for every column you want to display in the grid...<br>
<br>
jqgrid_pkg.save_grid_config (l_config);<br>
<br>
commit;<br>
<br>
end;<br>
<br>
</pre>


<i>Note:</i> It is also possible to render a grid from PL/SQL without saving the grid configuration in the database. However, this always results in a read-only grid (because the update process, being disconnected from the render process, needs to read the configuration from the database in order to do the update). See the example under "Usage", below.<br>
<br>
<h3>Table/Column Reference</h3>

The columns of the grid configuration tables are used as follows:<br>
<br>
<ul>
<li><b>JQGRID.APEX_APPLICATION_ID:</b> When an Apex application ID is specified, the user must be logged into Apex to update the grid</li>
<li><b>JQGRID.FOOTER_TEXT:</b> Text and/or HTML printed below the grid</li>
<li><b>JQGRID.GRID_HEIGHT:</b> Height of the grid, in pixels</li>
<li><b>JQGRID.GRID_MESSAGES:</b> Display status messages to user? (true, false)</li>
<li><b>JQGRID.GRID_NAME:</b> The name that identifies the grid. The name will also be used to generate various pieces of Javascript, so make sure the name does not contain any spaces or weird characters. You know what I'm talking about.</li>
<li><b>JQGRID.GRID_REMARKS:</b> Internal remarks, not displayed to the user</li>
<li><b>JQGRID.GRID_TITLE:</b> Title that will be displayed in the grid header</li>
<li><b>JQGRID.HEADER_TEXT:</b> Text and/or HTML printed above the grid</li>
<li><b>JQGRID.LOG_LEVEL:</b> Log level for grid (NONE, ERROR, WARN, INFO, DEBUG)</li>
<li><b>JQGRID.MULTI_SELECT:</b> Render the checkboxes to select rows? (true, false)</li>
<li><b>JQGRID.PUBLIC_API_BASE_URL:</b> The base URL to the PL/SQL package that handles DML operations, including the trailing slash. Only needed if you have set up alternate DADs in the PL/SQL gateway, leave blank in a default installation.</li>
<li><b>JQGRID.READ_ONLY:</b> If grid is read-only, editing is disabled (true, false)</li>
<li><b>JQGRID.ROW_UPDATE_API_RECTYPE:</b> Record type for row updates via API. This can be a user-defined PL/SQL record type, or a record type based on a table's %ROWTYPE. The record type must contain a field that matches the name of each editable column in the grid.</li>
<li><b>JQGRID.SCHEMA_NAME:</b> Schema of primary table for DML operations</li>
<li><b>JQGRID.TABLE_NAME:</b> Name of primary table for DML operations</li>
<li><b>JQGRID_COLUMN.COLUMN_NAME:</b> The column name, derived from the REF Cursor or SQL statement that populates the grid with data.</li>
<li><b>JQGRID_COLUMN.GRID_NAME:</b> Foreign key to grid definition</li>
<li><b>JQGRID_COLUMN.UPDATE_COLUMN_NAME:</b> Specify the actual column to update, if different from the displayed column. Relevant for columns which are displayed as select lists (combo boxes).</li>
</ul>


<h2>Usage</h2>

<i>Note:</i> These examples assume you have installed the contents of the "demo" folder into your database.<br>
<br>
<h3>Basic example</h3>

Add a page to your Apex application. Add a PL/SQL region to the page, and add the following code to the region:<br>
<br>
<pre>

begin<br>
<br>
-- required to include the necessary Javascript and CSS files<br>
jqgrid_pkg.include_static_files_once (p_path => 'path_to_root_folder_for_static_files_on_web_server');<br>
<br>
-- note: there are several ways to get a ref cursor<br>
-- here we are calling a function that returns a ref cursor<br>
-- other alternatives include using the CURSOR() function or the OPEN .. FOR statement<br>
<br>
jqgrid_pkg.render_grid ('my_product_info_grid', demo_product_pkg.get_products (p_category => 'Video') );<br>
<br>
end;<br>
<br>
</pre>

<h3>Render (read-only) grid from on-the-fly configuration (not stored in database)</h3>

<pre>

declare<br>
l_config jqgrid_pkg.t_grid_config;<br>
begin<br>
<br>
l_config.grid.grid_name := 'employees_custom';<br>
l_config.grid.grid_title := 'Testing jQGrid from PL/SQL !';<br>
l_config.grid.grid_height := 500;<br>
<br>
-- note: although the columns are set to be editable (on the client side),<br>
--       there will be no server-side processing<br>
--       unless the grid configuration is saved (persisted) to the database<br>
l_config.grid_columns(1).column_name := 'ENAME';<br>
l_config.grid_columns(1).display_name := 'Employee';<br>
l_config.grid_columns(1).editable := jqgrid_pkg.g_true;<br>
<br>
l_config.grid_columns(2).column_name := 'JOB';<br>
l_config.grid_columns(2).display_name := 'Job Position';<br>
l_config.grid_columns(2).editable := jqgrid_pkg.g_true;<br>
l_config.grid_columns(2).edit_type := 'select';<br>
l_config.grid_columns(2).list_of_values := 'STATIC:A:Master;B:Slave;C:Underslave';<br>
l_config.grid_columns(2).data_length := 200;<br>
<br>
l_config.grid_columns(3).column_name := 'HAS_MANAGER';<br>
l_config.grid_columns(3).display_name := 'Manager?';<br>
l_config.grid_columns(3).editable := jqgrid_pkg.g_true;<br>
l_config.grid_columns(3).edit_type := 'checkbox';<br>
<br>
<br>
-- required to include the necessary Javascript and CSS files<br>
jqgrid_pkg.include_static_files_once (p_path => 'path_to_root_folder_for_static_files_on_web_server');<br>
<br>
-- render the grid itself<br>
jqgrid_pkg.render_grid (l_config.grid.grid_name, p_sql => 'select empno as row__id, ename, job, ''Y'' as has_manager from emp', p_grid_config => l_config);<br>
<br>
end;<br>
<br>
</pre>

<h2>Static files</h2>

These are the static files (stylesheets and Javascript files) that must be included on a page for the grid to be rendered correctly.<br>
<br>
You can include this HTML code either by<br>
<ul><li>including the HTML code below in your Apex page template (changing the path as appropriate to your web server)<br>
</li><li>calling the procedure JQGRID_PKG.INCLUDE_STATIC_FILES_ONCE in a PL/SQL region (passing as a parameter to the procedure the path as appropriate to your web server).</li></ul>

<pre>

<link rel="stylesheet" type="text/css" media="screen" href="/i/muledev/devtest/jqgrid/css/redmond/jquery-ui-1.7.2.custom.css" /><br>
<link rel="stylesheet" type="text/css" media="screen" href="/i/muledev/devtest/jqgrid/css/ui.jqgrid.css" /><br>
<script type="text/javascript" src="/i/muledev/devtest/json2.js"><br>
<br>
Unknown end tag for </script><br>
<br>
<br>
<script type="text/javascript" src="/i/muledev/devtest/jqgrid/js/jquery-1.3.2.min.js"><br>
<br>
Unknown end tag for </script><br>
<br>
<br>
<script type="text/javascript" src="/i/muledev/devtest/jqgrid/js/i18n/grid.locale-en.js"><br>
<br>
Unknown end tag for </script><br>
<br>
<br>
<script type="text/javascript" src="/i/muledev/devtest/jqgrid/js/jquery.jqGrid.min.js"><br>
<br>
Unknown end tag for </script><br>
<br>
<br>
<br>
</pre>

Note that the above references a specific jQuery version. If you are already using a different (newer) version of jQuery on your web page, you should modify (remove) the references accordingly to avoid conflicts.<br>
<br>
<br>
<h2>Known Issues</h2>

<ul><li>None</li></ul>

<h2>Change History</h2>

<ul><li>See changelog.txt file