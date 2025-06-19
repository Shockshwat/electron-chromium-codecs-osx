This is the guide to build electron with codecs for intel macs. 

Everything remains the same as arm build until Step 4. The ffmpeg-codec.patch is obsolete here, You need ffmpeg-codec-intel.patch , This patch is also experimental but it should get you through most stuff. So instead of using ffmpeg-codec.patch you would be using this one and the rest of the guide remains more or less the same. 

If you get ffmpeg errors when using ninja build, You need to now manually edit ffmpeg-generated.gni file in src/third_party/ffmpeg/ in your electron dump to fix them. You will get errors like : "Unspecified symbol ff_ac3_parse_header" or "Undeclared symbol ff_convert" and the like, The way you fix them is first open the ffmpeg directory in an IDE like VS Code, and use the search function to find the undefined symbols.

For example, if I would've gotten the error : "Unspecified symbol ff_some_codec" then I would search the ffmpeg directory for "ff_some_codec" and when I find its declaration (usually int ff_some_codec or const ff_some_codec or the like, it will never be in an if statement or "err= " statement) I will first check if there is a condition on it (Let's say something like "#CONFIG-CODEC-ENABLED" is on top of the function). If there is a condition #CONFIG-CODEC-ENABLED then I will check if the flag CONFIG-CODEC-ENABLED is set to 1 in config.h and config_components.h (They are found in chromium/config/Chrome/mac/x64) If the flag is set to 0 I will set it to 1 and I will move to the next step, If it is already 1 then I will regardless move on to the next step. 

The next step would be see if the file is included in the ffmpeg build args, for which we would open ffmpeg-generated.gni, Now we need to be a little careful and need to remember if we changed the CONFIG flag or not, If the config was set to 1 by default (Or not present at all) then we can safely look for an if statement which matches our case which is (is_apple && current_cpu == "x64") and add the file which had the declaration of the missing symbol. Note that if the source file is ending with .c then it must go with ffmpeg_c_sources and if the file is ending with .asm then it will go in ffmpeg_asm_sources. 

If the CONFIG was set to 0 and we changed it to 1, We can either :

1. Try building again, if it fails then add the file in ffmpeg-generated.gni. 
2. Search for the file in ffmpeg-genrated.gni and check if the if statements above match our case. 

Either way the error will be gone once you do this and a new one would pop up which would require similar treatment (low chances, i tried my best to not get any errors at all). If no other errors pop up then well just leave the machine alone it will build the dist over time.

Do not forget to run gn command before ninja command to rebuild the targets.
