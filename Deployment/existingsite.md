#Migrating an Existing Site to Umbraco as a Service
Sometimes you may already have an umbraco site built that did not start with a clone of an Umbraco as a Service site. Or perhaps you have decided to move a site that's already live to Umbraco as a Service. In any case, migrating an existing site is not difficult, but it does require some specific steps and an understanding of how Umbraco as a Service deployments work can be very helpful.

With that in mind, here are the steps to take.

##Understanding what you have
Prior to undertaking a migration you'll want to make sure you know the packages, add-ons, and custom code your site is using.  This is especially important if you may be using custom property editors that will require data resolvers in order to work properly with the Umbraco as a Service deployment engine. Some common property editors that will require a data resolver are; [Archetype](https://github.com/leekelleher/Archetype.Courier), [Mortar](https://github.com/leekelleher/umbraco-mortar/tree/develop/Src/Our.Umbraco.Mortar.Courier), and [Nested Content](https://github.com/leekelleher/umbraco-nested-content) which does not currently contain a data resolver and will not deploy properly. There are certainly other property editors that will require a custom data resolver but, for the most part, property editors that store data as umbraco data will deploy without requiring any special attention.

If you have used Courier with your site previously and deployments work as expected, then you can be relatively certain it will also deploy properly with Umbraco as a Service.

##Step-by-step
1. Make sure your existing site is upgraded to the latest released version of umbraco. You can find detailed information about upgrading your site [here](https://our.umbraco.org/documentation/Getting-Started/Setup/Upgrading/). You'll also want to make sure the Umbraco as a Service site you create and clone in step 5 is the same version.
2. Verify your site runs without errors.  
    * Hint: check the umbracoTraceLog.txt log file.
    * Ideally your site will run locally using the SqlCe database as this will make content migration easier. Don't worry if that's not possible, you can still complete the migration.
3. With the site running empty the Content and Media Recycle bins.
4. Shut down the site and delete the following files and folders:
    * `/app_data/TEMP`
    * `/app_data/logs`
    * `/app_data/cache`
    * `/app_data/preview`
    * `/app_data/umbraco.config`
    * `/data/[*]`
5. Create a new Umbraco as a Service project and clone the Dev site to your local machine.
6. Verify the existing site and the cloned Umbraco as a Service site are the same umbraco version
7. Copy all folders from the existing site to the Umbraco as a Service site making sure to leave all the files in the folders as the uUmbraco as a Service versions:
    * `/bin/`
    * `/data/backoffice/users/`
    * `/umbraco/`
    * `/umbraco_client/`
8. Merge the config files as needed paying special attention to
    * `web.config`
    * `/config/courier.config`
    * `/config/dashboard.config`
    * `/config/umbracodeploy.config`
9. If using SqlCe then also make sure the SqlCe database from your existing site replaces the current SqlCe file - `/app_data/umbraco.sdf`
    * If using a local Sql Server make sure the connection string in web.config is set correctly.
    * If you have a backoffice user that has the exact same name as the user for Umbraco as a Service you will need to change to user name in the database at this time or update the password before deploying via the backoffice.
    * In the `/data/backoffice/users/` folder you will have user files that correspond to each user that has been created via the Umbraco as a Service portal.  To add any of these users to the site simply rename the files by removing the leading underscore.
10. Open a command prompt at the `/data/` folder and add a deploy marker.
    * data> echo > deploy
11. Now run the local Umbraco as a Service site with the updated files and database. The deploy engine will start when the deploy marker is detected.  This will likely complete relatively quickly. This is adding the users created by Umbraco as a Service to your existing database.
12. Once complete verify your site has all meta data, content, and media as expected.
13. Delete the folder `/data/Revision/` from the file system.  
14. In the same browser session as the logged in umbraco user open a new tab at the address:  
    * `http://localhost:port/umbraco/backoffice/api/CourierAdmin/Rebuild`
    * That will create a serialized version of all your site's meta data and will display the following in the browser:
    * `<string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">Rebuilding</string>`
15. At this point you are ready to deploy your site to Umbraco as a Service - yay!
16. Make sure you git add and commit all the files you added to the site including the serialized files in `/data/Revison/` but excluding `/media/`.  This should be set correctly by the included .gitignore file.
17. Git push your commit to your Umbraco as a Service dev site checking that the "Deployment Complete" message is displayed.
    * If you have a very large commit to push, you may need to configure your git client for this.  
    * Use: git config `http.postBuffer 524288000`
18. Verify all meta data is in place as expected in your Umbraco as a Service dev site
    * `http://dev-your-project-name.s1.umbraco.io/umbraco/`
19. Once the meta data is in place, you can use the standard Umbraco as a Service content deployment approach to deploy your content and media from local to Dev.  Using the "Queue for Transfer" and the Deploy Dashboard.
    * Note: if you have a very large amount of content and or media you may have the best result in deploying content and media independently.
