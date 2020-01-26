---
layout: post
title: 'Janus: Refactoring a Plugin'
---
The sample Janus Plugins are written in C and they are very good.  
But, it's not very readable.  I'd like a higher level description of what the sample plugin app does.  Though the documentation is good, I still want a complete, high level description.  I'll do this by refactoring, which will also allow me to take out reusable snippets.
  
Particularly looking to refactor to improve:  separation of concerns  (single responsibility) .  Make it more obvious where the configuration file comes from.
  
# Pre- Notes
1.  Because these plugins are .so shared modules, I should be able to build the plugin by itself, then test with a real Janus instance.  Janus' build script builds the sample plugins with it, but that's just for convenience.  As a product developer, what I would do is have a separate build environment, git repo, etc. for my new plugin, build it, then deploy it to the /path/to/janus/lib/janus/plugins/  (I believe that's where you deploy them?)  
2.  Oh crap, it's built with autotools, I forgot.  I have no idea, and no desire to learn autotools.  Hmm how do I build the plugin?
3.  Ah yes, thank you https://github.com/mquander/janus-helloworld-plugin .  I will fork it!  Then copy over the recordplaytest demo!
4.  Ah, it didn't include any Janus headers, so those must come from the Janus installation on the server.  I don't have Janus installed on my laptop. Let's just make sure by attempting a build on my laptop.   Need jansson.  Ah yes, of course, plugins/plugin.h not found.  Let's make sure that exists on the server installation.  Well it's actually in /path/to/janus/include/janus/plugins .  Ah, checking the gcc -I flags, yes, it's in there.  Good!  Eh, let's build it on the cloud and then hook up a Jenkins later.
5.  Cool, it builds on my server that has Janus.  Let's... see if make install followed by restarting Janus will pick up the new plugin!
```
JANUS AudioBridge plugin initialized!
Loading plugin 'libjanus_helloworld.so'...
JANUS hello world plugin initialized!
Loading plugin 'libjanus_recordplay.so'...
JANUS Record&Play plugin initialized!
```
# Yay
6.  Quick side note, to clean up the build I did a `git clean -f -d -x`
7.  Now I can figure out how to change the name of the plugin and copy in the source from recordplaytest.  Well, doing a `grep -ir hello *` , excluding janus_helloworld.c , only shows a few matches.  That's cool, the name is not significant.  OK, it worked, but there are still "hello world" occurences in the source itself.
8.  Before I copy paste the source from the recordplay plugin, let's look at that `grep -i hello *` again.  Lots of matches, but which of those actually matter?  The #defines at the top, eg. `#define JANUS_HELLOWORLD_PACKAGE "janus.plugin.helloworld"` don't need to change, because they are simply returned by the getters.  In other words, I can copy paste the source from recordplay_plugin, the names won't be HELLO anymore, they'll be RECORDPLAY, and this is OK.
The interface implementation, `janus_plugin` is a struct of function pointers, so the names don't matter, they're just function pointers.  The symbol name of that implementation is `janus_helloworld_plugin` , ie. `static janus_plugin janus_helloworld_plugin` .  A pointer to this struct is returned by the `*create()` function.  
So, my conclusion is, I will only have to change the name of `janus.plugin.recordplay` or whatever it is, to something else, so the plugin namespace does not collide.  
9.  Now I can copy paste the source.  Ah crap.  There are headers in janus-gateway repo that have to be included in the build.  I don't know how to do that in autotools.  Dang!   Oh, looking at Makefile.am , it appears I can specify a -I flag.  Oh, I don't need to go to the source, the headers are exported in the Janus install.  Oh, the headers directory is already there!  Problem was the `../blah.h` should be `blah.h` .  Changing that...  OK, I built it again with `make`, no need to do any more autotools stuff.  I get an error, plugin could not be initialized because no configuration file could be read.  Let's copy one in there... `/path/to/janus/etc/janus/`  Yay, now it works!
  
# Beginning to understand the plugin
The first function definitions are the static helper functions, which is OK convention.  The first plugin implementation function is `init`.  This is called by Janus when Janus starts.  I saw this before, when Janus failed to start because a plugin couldn't be initialized.  `init` first reads the configuration file, which is assumed to have `"%s/%s.cfg" format.  The path to the config files is an argument to `init()` , which somehow must come from Janus.  The config file includes a path to where the recordings are stored.  That path is stored in a variable.  `notify_events` is a static boolean, which is checked later before doing `gateway->notify_event` for sending an event.  
2.  Now, `recordings = g_hast_table_new_full` , `sessions = g_hst_table_new` , `messages = g_async_queue_new_full` .
3.  `handler_thread = g_thread_try_new` starts a thread with function `janus_recordplay_handler`  .  Oh god, that function is a monster, 421 lines, tons of if statements.  

# For reference, here is the plugin interface definition
```
/* Plugin setup */
static janus_plugin janus_recordplay_plugin =
  JANUS_PLUGIN_INIT (
    .init = janus_recordplay_init,
    .destroy = janus_recordplay_destroy,

    .get_api_compatibility = janus_recordplay_get_api_compatibility,
    .get_version = janus_recordplay_get_version,
    .get_version_string = janus_recordplay_get_version_string,
    .get_description = janus_recordplay_get_description,
    .get_name = janus_recordplay_get_name,
    .get_author = janus_recordplay_get_author,
    .get_package = janus_recordplay_get_package,
    
    .create_session = janus_recordplay_create_session,
    .handle_message = janus_recordplay_handle_message,
    .setup_media = janus_recordplay_setup_media,
    .incoming_rtp = janus_recordplay_incoming_rtp,
    .incoming_rtcp = janus_recordplay_incoming_rtcp,
    .incoming_data = janus_recordplay_incoming_data,
    .slow_link = janus_recordplay_slow_link,
    .hangup_media = janus_recordplay_hangup_media,
    .destroy_session = janus_recordplay_destroy_session,
    .query_session = janus_recordplay_query_session,
  );
```
```
int janus_recordplay_init(janus_callbacks *callback, const char *onfig_path);
void janus_recordplay_destroy(void);
int janus_recordplay_get_api_compatibility(void);
int janus_recordplay_get_version(void);
const char *janus_recordplay_get_version_string(void);
const char *janus_recordplay_get_description(void);
const char *janus_recordplay_get_name(void);
const char *janus_recordplay_get_author(void);
const char *janus_recordplay_get_package(void);
void janus_recordplay_create_session(janus_plugin_session *handle, int *error);
struct janus_plugin_result *janus_recordplay_handle_message(janus_plugin_session *handle, char *transaction, json_t *message, json_t *jsep);
void janus_recordplay_setup_media(janus_plugin_session *handle);
void janus_recordplay_incoming_rtp(janus_plugin_session *handle, int video, char *buf, int len);
void janus_recordplay_incoming_rtcp(janus_plugin_session *handle, int video, char *buf, int len);
void janus_recordplay_incoming_data(janus_plugin_session *handle, char *buf, int len);
void janus_recordplay_slow_link(janus_plugin_session *handle, int uplink, int video);
void janus_recordplay_hangup_media(janus_plugin_session *handle);
void janus_recordplay_destroy_session(janus_plugin_session *handle, int *error);
json_t *janus_recordplay_query_session(janus_plugin_session *handle);
```
Ah very nice, these function declarations have the same order as the struct def.  
Now I feel bad for saying it was unreadable.  
1.  Notice, the `init()` function takes a callback struct and a config path.  The other functions eg. `incoming_rtp` take a `janus_plugin_session`.  Let's see what they are.
  
# What is janus_plugin_session ? 
Notice there is  `void janus_recordplay_create_session(janus_plugin_session *handle, int *error);` .  This is called by Janus and implemented by the plugin.  What does it do in recordplay plugin?  Here it is
```
void janus_recordplay_create_session(janus_plugin_session *handle, int *error) {
  if(g_atomic_int_get(&stopping) || !g_atomic_int_get(&initialized)) {
    *error = -1;
    return;
  } 
  janus_recordplay_session *session = g_malloc0(sizeof(janus_recordplay_session));
  session->handle = handle;
... populate session fields
  janus_mutex_init(&session->rec_mutex);
  session->destroyed = 0;
  g_atomic_int_set(&session->hangingup, 0);
... populate session fields
  handle->plugin_handle = session;
  janus_mutex_lock(&sessions_mutex);
  g_hash_table_insert(sessions, handle, session);
  janus_mutex_unlock(&sessions_mutex);

  return;
}
```
This is convoluted, I don't get it.  What's happening here is, the recordplay plugin has a hash table called `sessions`.  A `janus_recordplay_session` is malloc'd , populated, then stored in the hash table by reference.  

Let's take a look at `janus_recordplay_incoming_rtp` , 
