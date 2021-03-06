When you provision a MongoLab database, MongoLab transmits a connection URI to Azure in MongoDB's standard connection string format. This value is used to initiate a MongoDB connection through your choice of MongoDB driver. For more information about connection strings, see [Connections](http://www.mongodb.org/display/DOCS/Connections) at mongodb.org.

**This URI contains your database user name and password.  Treat it as sensitive information and do not share it.**

You can retrieve this URI in the Azure Portal using the following steps:

1. Select **Add-ons**.  
   ![AddonsButton](./media/howto-get-connectioninfo-mongolab/button-addons.png)
2. Locate your MongoLab service in your add-on list.  
   ![MongolabEntry](./media/howto-get-connectioninfo-mongolab/entry-mongolabaddon.png)
3. Cick the name of your add-on to reach the add-on page.
4. Click **Connection Info**.  
   ![ConnectionInfoButton](./media/howto-get-connectioninfo-mongolab/button-connectioninfo.png)  
   Your MongoLab URI displays:  
   ![ConnectionInfoScreen](./media/howto-get-connectioninfo-mongolab/dialog-mongolab_connectioninfo.png)  
5. Click the clipboard button to the right of the MONGOLAB_URI value to copy the full value to the clipboard.

[entry-mongolabaddon]: ./media/howto-get-connectioninfo-mongolab/entry-mongolabaddon.png
[button-connectioninfo]: ./media/howto-get-connectioninfo-mongolab/button-connectioninfo.png
[screen-connectioninfo]: ./media/howto-get-connectioninfo-mongolab/dialog-mongolab_connectioninfo.png
[button-addons]: ./media/howto-get-connectioninfo-mongolab/button-addons.png
