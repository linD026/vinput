#
# VINPUT
# virtual input device layer
# Tristan Lelong <tristan.lelong@blunderer.org>
#

1) Kernel API
-------------
vinput is a API to allow easy development of virtual input drivers.

The drivers needs to export a vinput_devince function that contains the virtual device name and vinput_ops structure that describes:
- the init function: init
- the input event injection function: send
- the readback function: read

Then using vinput_register_device and vinput_unregister_device will add a new device to the list of support virtual input devices.

int init(struct vinput *);
  This function is passed a struct vinput already initialized with an allocated struct input_dev. The init function is responsible for initializing the
  capabilities of the input device and register it.

int send(struct vinput *, char *, int);
  This function will receive a user string to interpret and inject the event using the input_report_XXXX or input_event call.
  The string is already copied from user.

int read(struct vinput *, char *, int);
  This function is used for debugging and should fill the buffer parameter with the last event sent in the virtual input device format.
  The buffer will then be copied to user.


2) Userland API:
----------------
vinput devices are created and destroyed using sysfs.
event injection is done thru a /dev node.

The device name will be used by the userland to export a new virtual input device.

To create a vinputX sysfs entry and /dev node.
	$ echo "vkbd" > /sys/class/vinput/export

To unexport the device, just echo its id in unexport:
	$ echo "0" > /sys/class/vinput/unexport

3) VKBD:
--------
This is the virtual keyboard. It supports all KEY_MAX keycodes. The injection format is the KEY_CODE such as defined in linux/input.h.
A positive value means KEY_PRESS while a negative value is a KEY_RELEASE.
The keyboard supports repetition when the key stays pressed for too long.

ex: simulate a key press on "g" (KEY_G = 34 )
	$ echo "+34" > /dev/vinput0

ex: simulate a key release on "g" (KEY_G = 34 )
	$ echo "-34" > /dev/vinput0

