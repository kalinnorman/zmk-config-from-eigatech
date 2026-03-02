# Details

This is a repository that contains a large number of zmk configurations for different keyboards. I've forked it from Eigatech and originally set the totem-dongle as the default branch, but this is a newer branch that I created with the intent of switching from a fully bluetooth (with a dongle) setup to a partially wired setup between the central and computer, and part bluetooth -- between the central and peripheral.

To alter the keymap please only update the config/totem.keymap file, but other changes have been made from the totem-dongle branch in order to remove the dongle and adjust how the halves talk to each other and the computer.

If this branch is set as the main branch, then any time that a new change is pushed to GitHub it will trigger a workflow that will build the necessary files. To flash the keyboard halves:
1. Navigate to the Actions tab on the repository webpage
2. Click on the most recent workflow run
3. Scroll down to Artifacts and download the firmware zip file
4. Extract the contents
5. For each microcontroller:
    1. Ensure that the microcontroller is not being powered by another device (turn the two halves of the keyboard off)
    2. Connect the microcontroller to the computer with a usb cable
    3. Double click the reset button (either through the keyboard reset button that is connected or the button directly on the microcontroller itself)
    4. A new storage device should now be available on the computer -- If it isn't then try the reset button again or try a new usb cable, as not all cables are able to transfer data
    5. Copy and paste the settings_reset configuration into the microcontroller's storage
        - NOTE: This is necessary to use the settings_reset config to wipe everything if any changes have been made to anything other than the keymap, like if switching between the use of a dongle and not, or other non-keymap related changes, but if only the config/totem.keymap file has been changed, then it should be okay to skip flashing the settings_reset file and go straight to the left or right config.
    6. Wait for the copying to finish, after which the storage device should disappear
    7. Repeat steps 1 - 4
    5. Copy and paste the appropriate configuration into the microcontroller's storage (the configurations are all in the extracted folder from before, there should be one for the left half and one for the right half of the keyboard)
    6. After the contents finish copying over, the storage device should disappear, and you can disconnect the microcontroller from the computer
    7. It's flashed!
