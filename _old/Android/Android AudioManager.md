> 源码基于 Android 11 API 30







AudioManager.STREAM

```
AudioManager.STREAM_VOICE_CALL 0 通话
AudioManager.STREAM_SYSTEM 1
AudioManager.STREAM_RING 2 铃声
AudioManager.STREAM_MUSIC 3  媒体，音乐
AudioManager.STREAM_ALARM 4 闹钟
AudioManager.STREAM_NOTIFICATION 5 通知
AudioManager.STREAM_DTMF 8 双音多频
AudioManager.STREAM_ACCESSIBILITY 10

//e.g.
audioManager.setStreamVolume(AudioManager.STREAM_MUSIC, now, AudioManager.FX_KEY_CLICK);
```



AudioManager.FLAG

```
AudioManager.FLAG_ALLOW_RINGER_MODES
AudioManager.FLAG_PLAY_SOUND
AudioManager.FLAG_REMOVE_SOUND_AND_VIBRATE
AudioManager.FLAG_SHOW_UI
AudioManager.FLAG_VIBRATE

//e.g.
audioManager.setStreamVolume(streamType, index, AudioManager.FLAG_SHOW_UI);
```



AudioManager.MODE

```
AudioManager.MODE_CALL_SCREENING 4 呼叫筛选中（呼叫过滤）
AudioManager.MODE_CURRENT -1
AudioManager.MODE_INVALID -2
AudioManager.MODE_IN_CALL 2 
AudioManager.MODE_IN_COMMUNICATION 3
AudioManager.MODE_NORMAL 0 普通（不响铃，不呼叫）
AudioManager.MODE_RINGTONE 1 铃声

//e.g.
audioManager.setMode(AudioManager.MODE_IN_COMMUNICATION);
```



AudioService 里系统定义的默认音量

```java
/** Maximum volume index values for audio streams */
    protected static int[] MAX_STREAM_VOLUME = new int[] {
        5,  // STREAM_VOICE_CALL
        7,  // STREAM_SYSTEM
        7,  // STREAM_RING
        15, // STREAM_MUSIC
        7,  // STREAM_ALARM
        7,  // STREAM_NOTIFICATION
        15, // STREAM_BLUETOOTH_SCO
        7,  // STREAM_SYSTEM_ENFORCED
        15, // STREAM_DTMF
        15, // STREAM_TTS
        15, // STREAM_ACCESSIBILITY
        15  // STREAM_ASSISTANT
    };

    /** Minimum volume index values for audio streams */
    protected static int[] MIN_STREAM_VOLUME = new int[] {
        1,  // STREAM_VOICE_CALL
        0,  // STREAM_SYSTEM
        0,  // STREAM_RING
        0,  // STREAM_MUSIC
        1,  // STREAM_ALARM
        0,  // STREAM_NOTIFICATION
        0,  // STREAM_BLUETOOTH_SCO
        0,  // STREAM_SYSTEM_ENFORCED
        0,  // STREAM_DTMF
        0,  // STREAM_TTS
        1,  // STREAM_ACCESSIBILITY
        0   // STREAM_ASSISTANT
    };
```

