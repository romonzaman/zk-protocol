# Data Operations #

[Go to Main Page](../protocol.md)

Author: Alexander Marin <alexanderm2230@gmail.com>

[TOC]

## Current Descriptions ##

Here is a list of SDK functions, from **Data-User.h** and **Data-Record** files, that shows which functions can be replicated with the current spec:

|SDK function name	|Described(X=Yes, O=No)	|Notes							|
|---			|:---:			|---							|
|ReadAllUserID		|**X**			|							|
|GetAllUserID		|**O**			|Applicable only to BW.					|
|GetAllUserInfo		|**O**			|Applicable only to BW.					|
|EnableUser		|**O**			|Applicable only to BW.					|
|SSR_EnableUser		|**X**			|							|
|ModifyPrivilege	|**O**			|Applicable only to BW.					|
|SetUserInfo		|**O**			|Applicable only to BW.					|
|GetUserInfo		|**O**			|Applicable only to BW.					|
|SetUserInfoEx		|**X**			|							|
|GetUserInfoEx		|**X**			|							|
|DeleteUserInfoEx	|**O**			|Todo.							|
|SSR_GetAllUserInfo	|**X**			|Operates on the info obtained with ReadAllUserID.	|
|SSR_GetUserInfo	|**X**			|Operates on the info obtained with ReadAllUserID.	|
|SSR_SetUserInfo	|**X**			|							|
|GetUserInfoByPIN2	|**O**			|Applicable only to BW.					|
|GetUserInfoByCard	|**O**			|Applicable only to BW.					|
|GetUserIDByPIN2	|**O**			|Applicable only to BW.					|
|GetPIN2		|**O**			|Applicable only to BW.					|
|GetEnrollData		|**O**			|Applicable only to BW.					|
|SetEnrollData		|**O**			|Applicable only to BW.					|
|DeleteEnrollData	|**O**			|Applicable only to BW.					|
|SSR_DeleteEnrollData	|**O**			|					|
|SSR_DeleteEnrollDataExt|**O**			|					|
|GetEnrollDataStr	|**O**			|Applicable only to BW.					|
|SetEnrollDataStr	|**O**			|Applicable only to BW.					|
|ReadAllTemplate	|**O**			|					|
|DelUserTmp		|**O**			|Applicable only to BW.					|
|SSR_DelUserTmp		|**O**			|						|
|SSR_SetUserTmpExt	|**O**			|Applicable only to BW.					|
|GetUserTmp		|**O**			|Applicable only to BW.					|
|SetUserTmp		|**O**			|Applicable only to BW.					|
|GetUserTmpStr		|**O**			|Applicable only to BW.					|
|SetUserTmpStr		|**O**			|Applicable only to BW.					|
|GetUserTmpEx		|**O**			|					|




## Read All User IDs ##

With this procedure the users info can be obtained, except the fingerprint templates.

First disable the device:

	> packet(id=CMD_DISABLEDEVICE)
		> packet(id=CMD_ACK_OK)

Then send a command with the id `CMD_DATA_WRRQ` and with a fixed payload, field description for this payload it is still unknown.

	packet(id=CMD_DATA_WRRQ, data=0109000500000000000000)

Depending of the size of the users info structure, the device may send this info in two ways:

1.For "small" structures, the machine would send the info structure immediately

	> packet(id=CMD_DATA_WRRQ, data=0109000500000000000000)
		> packet(id=CMD_DATA, data=<users info>)

2.For bigger structures see the [Exchange of Data](ex_data.md) spec.

The fields of the `users info` structure are given in the following table:

|Name		|Description				|Value[hex]	|Size[bytes]	|Offset		|
|---		|---					|---		|---		|---		|
|size users info|Total size of user info entries.	|N*72 (<)	|2		|0		|
|zeros		|Null bytes.				|00 00		|2		|2		|
|user1 entry	|Info of user 1.			|varies		|72		|4		|
|user2 entry	|Info of user 2.			|varies		|72		|76		|
|...		|...					|varies		|72		|...		|
|userN entry	|info of user N.			|varies		|72		|info_size-72+4	|

(<): Little endian format.

The contents of each user entry, are shown in the next table:

|Name			|Description							|Value[hex]						|Size[bytes]	|Offset	|
|---			|---								|---							|---		|---	|
|user sn		|Internal serial number for the user.				|varies (<)					|2		|0	|
|permission token	|Sets permission for the given user and carries enable flag.	|varies							|1		|2	|
|password		|User password, stored as a string.				|varies							|8		|3	|
|name(*)		|User's name.							|varies							|24		|11	|
|card number		|User's card number, stored as int.				|varies (<)						|4		|35	|
|			|Fixed.								|01 00 00 00 00 00 00 00 00				|9		|39	|
|user id		|User ID, stored as a string.					|varies							|9		|48	|
|fixed			|								|00 00 00 00 00 00 00 00 00 00 00 00 00 00 00		|15		|57	|

(<): Little endian format.
(*): The name string should be terminated with the null char `\x00`, so the allowed size for user name is really 23 chars.

The permission token defines the permissions of the user and the also sets the state of the user.

|Bit Offset	|7-4	|3	|2	|1	|0	|
|---		|---	|---	|---	|---	|---	|
|--		|Unused	|P2	|P1	|P0	|E0	|

The number given by `P2P1P0` indicates the user admin level:

|P2P1P0	|Level		|
|---	|---		|
|000	|Common user	|
|001	|Enroll user	|
|011	|Admin		|
|111	|Super admin	|

The `E0` bit only enables/disables the user

|E0	|State		|
|---	|---		|
|0	|Eneabled	|
|1	|Disabled	|

Finally send the enable device command to put the device in normal operation:

	> packet(id=CMD_ENABLEDEVICE)
		> packet(id=CMD_ACK_OK)

## Enable User ##

Same as Set User Info procedure, in this case just the bit `E0` should be changed.

## Set User Verification Mode ##

To change the verification style of a given user, use the `CMD_VERIFY_WRQ` command, this packet should be sent with the new verification style, using a specific codification.

	> packet(id=CMD_VERIFY_WRQ, data=<verify info>)
		> packet(id=CMD_ACK_OK)

Where the verify info structure, is 24 bytes long and has the following fields:

|Name			|Description					|Value[hex]	|Size[bytes]	|Offset	|
|---			|---						|---		|---		|---	|
|user sn		|Internal serial number for the user.		|varies (<)	|2		|0	|
|verification mode	|Verification mode to be used, see next table.	|varies		|1		|2	|
|zeros			|Fixed.						|zeros		|21		|3	|

(<): Little endian format.

|Verification Mode(x)	|Value[base 10]	|Value[hex]	|
|---			|---		|---		|
|Group Verify		|0		|0		|
|FP+PW+RF		|128		|80		|
|FP			|129		|81		|
|PIN			|130		|82		|
|PW			|131		|83		|
|RF			|132		|84		|
|FP+PW			|133		|85		|
|FP+RF			|134		|86		|
|PW+RF			|135		|87		|
|PIN&FP			|136		|88		|
|FP&PW			|137		|89		|
|FP&RF			|138		|8a		|
|PW&RF			|139		|8b		|
|FP&PW&RF		|140		|8c		|
|PIN&FP&PW		|141		|8d		|
|FP&RF+PIN		|142		|8e		|

(x): The symbol "+" is used as a logic **or**, that means the verification may be performed in either way, while the symbol "&" is used as logic **and**, that means the verication needs both methods to be accepted.

## Get User Verification Mode ##

To get the verification style of a given user, use the `CMD_VERIFY_RRQ` command.

	> packet(id=CMD_VERIFY_RRQ, <user sn>)
		> packet(id=CMD_ACK_OK, data=<verify info>)

Where the user sn identifies the user, the value is stored in a 2 byte field in little endian format.

The `verify info` is the same structure shown in the previous section.

## Set User Info ##

This procedure is used to modify info of existing users, if the user doesn't exist, then it will be created.

Here is a list of the parameters that may be changed with this procedure:

- User ID.
- User name.
- User password.
- User admin level.
- Enable state.

The idea is to send a new user entry to overwrite the previous user data, to do this first disable the device, use the `CMD_USER_WRQ` command to send the new user entry, which has the same fields shown in the "Read All User IDs" section. Finally refresh the data and enable the device.

	> packet(id=CMD_DISABLEDEVICE)
		> packet(id=CMD_ACK_OK)
	> packet(id=CMD_USER_WRQ, data=<new entry>)
		> packet(id=CMD_ACK_OK)
	> packet(id=CMD_REFRESHDATA)
		> packet(id=CMD_ACK_OK)
	> packet(id=CMD_ENABLEDEVICE)
		> packet(id=CMD_ACK_OK)


[Go to Main Page](../protocol.md)