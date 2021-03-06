Mugo ESI
==================

What is it?
-----------

Mugo ESI is a framework to enable you to embed ESI widgets into your eZ Publish 4.x / legacy website. These widgets will work behind Varnish or another reverse proxy that supports ESI. If you are accessing the site without Varnish for testing purposes, it will load the widget using Ajax in order to ensure that it is loaded dynamically. In the future, we might add support to make a direct module view call.

In short, Mugo ESI comes with a template operator that will detect, based on the presence of a Surrogate-Capability request header, whether to support ESI.

Your actual ESI widget is a module view following the pattern: /esiwidget/view/(widget)/widget_name

The TTL of your widget can be set in your custom PHP function or in an INI file.

How to use it?
--------------

There is an example ESI widget "timestamp_example" included in the extension.

Supposing that you want to create a new widget that just shows the current user's name, you would do the following:

Create mugo_esi.ini.append.php in your own extension and add:

    [Widget_username]
    ClassName=MyESIWidgets
    FunctionName=usernameAction
    TTL=0

Create extension/yourextension/classes/myesiwidgets.php:

    <?php
    /*
     * A class containing functions for each dynamic widget
     *
    */
    class MyESIWidgets
    {
        /*
         * Return a single array containing all of the data you need as array elements
         * You can set the TTL with an element 'ttl' and a value in seconds
        */
        public static function usernameAction()
        {
            $user = eZUser::currentUser();
            $username = $user->attribute( 'contentobject' )->attribute( 'name' );
            return array( 'username' => $username );
        }
    }


Create extension/myextension/design/standard/templates/esiwidget/widgets/username.tpl ("username" coming from the name of your widget in the INI file) and output the username:

    {$data.username|wash()}

Use your new widget! In the relevant template, call the widget:

    {mugo_esi_widget( 'username' )}

You can pass parameters to the esi widget and they should be passed as GET parameters since we support multiple, here is one example on how to pass the param 'tagID' to the widget:

    {mugo_esi_widget( 'paramtest', hash( 'tagID', $tagID ) )}

And you can read the parameter using the eZHTTPTool class:

    <?php
    class MyESIWidgets
    {
        public static function paramtestAction()
        {
            $tagID = eZHTTPTool::hasGetVariable('tagID') ? eZHTTPTool::getVariable('tagID') : false;
            return array('tag_id' => $tagID);
        }
    }

You must include the following code in your pagelayout in order to support ESI widgets in full view templates. This is to ensure that eZ Publish will pass the HTTP header informing Varnish that there are ESI tags in the response.:

{if is_set( $module_result.content_info.persistent_variable.surrogate )}
    {set_header_variable( 'Surrogate-Control', 'abc=ESI/1.0' )}
{/if}
