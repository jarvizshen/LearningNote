创建通知


如需详细了解通知的各个部分，参阅通知剖析。
设置通知内容

首先，需要使用 NotificationCompat.Builder 对象设置通知内容和渠道。以下示例显示了如何创建包含下列内容的通知：

    小图标，通过 setSmallIcon() 设置。这是所必需的唯一用户可见内容。
    标题，通过 setContentTitle() 设置。
    正文文本，通过 setContentText() 设置。
    通知优先级，通过 setPriority() 设置。优先级确定通知在 Android 7.1 和更低版本上的干扰程度。（对于 Android 8.0 和更高版本，必须设置渠道重要性，如下一节中所示。）
    var builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle(textTitle)
            .setContentText(textContent)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    

注意，NotificationCompat.Builder 构造函数要求提供渠道 ID。这是兼容 Android 8.0（API 级别 26）及更高版本所必需的，但会被较旧版本忽略。

默认情况下，通知的文本内容会被截断以放在一行。如果想要更长的通知，可以使用 setStyle() 添加样式模板来启用可展开的通知。例如，以下代码会创建更大的文本区域：
    var builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Much longer text that cannot fit one line...")
            .setStyle(NotificationCompat.BigTextStyle()
                    .bigText("Much longer text that cannot fit one line..."))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    



必须先通过向 createNotificationChannel() 传递 NotificationChannel 的实例在系统中注册应用的通知渠道，然后才能在 Android 8.0 及更高版本上提供通知。因此以下代码会被 SDK_INT 版本上的条件阻止：
    private fun createNotificationChannel() {
        // Create the NotificationChannel, but only on API 26+ because
        // the NotificationChannel class is new and not in the support library
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val name = getString(R.string.channel_name)
            val descriptionText = getString(R.string.channel_description)
            val importance = NotificationManager.IMPORTANCE_DEFAULT
            val channel = NotificationChannel(CHANNEL_ID, name, importance).apply {
                description = descriptionText
            }
            // Register the channel with the system
            val notificationManager: NotificationManager =
                getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }
    

由于必须先创建通知渠道，然后才能在 Android 8.0 及更高版本上发布任何通知，因此应在应用启动时立即执行这段代码。反复调用这段代码是安全的，因为创建现有通知渠道不会执行任何操作。

NotificationChannel 构造函数需要一个 importance，它会使用 NotificationManager 类中的一个常量。此参数确定出现任何属于此渠道的通知时如何打断用户，但还必须使用 setPriority() 设置优先级，才能支持 Android 7.1 和更低版本（如上所示）。

虽然必须按本文所示设置通知重要性/优先级，但系统不能保证会获得提醒行为。在某些情况下，系统可能会根据其他因素更改重要性级别，并且用户始终可以重新定义指定渠道适用的重要性级别。

如需详细了解不同级别的含义，参阅通知重要性级别。
设置通知的点按操作

每个通知都应该对点按操作做出响应，通常是在应用中打开对应于该通知的 Activity。为此，必须指定通过 PendingIntent 对象定义的内容 Intent，并将其传递给 setContentIntent()。

以下代码段展示了如何创建基本 Intent，以在用户点按通知时打开 Activity：
    // Create an explicit intent for an Activity in your app
    val intent = Intent(this, AlertDetails::class.java).apply {
        flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
    }
    val pendingIntent: PendingIntent = PendingIntent.getActivity(this, 0, intent, 0)

    val builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Hello World!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            // Set the intent that will fire when the user taps the notification
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
    

此代码会调用 setAutoCancel()，它会在用户点按通知后自动移除通知。

以上所示的 setFlags() 方法可帮助保留用户在通过通知打开应用后的预期导航体验。但是否要使用这一方法取决于要启动的 Activity 类型，类型可能包括：

    专用于响应通知的 Activity。用户在正常使用应用时不会无缘无故想导航到这个 Activity，因此该 Activity 会启动一个新任务，而不是添加到应用的现有任务和返回堆栈。这就是以上示例中创建的 Intent 类型。
    应用的常规应用流程中存在的 Activity。在这种情况下，启动 Activity 时应创建返回堆栈，以便保留用户对返回和向上按钮的预期。

如需详细了解配置通知 Intent 的不同方法，参阅从通知启动 Activity。
显示通知

如需显示通知，调用 NotificationManagerCompat.notify()，并将通知的唯一 ID 和 NotificationCompat.Builder.build() 的结果传递给它。例如：
    with(NotificationManagerCompat.from(this)) {
        // notificationId is a unique int for each notification that you must define
        notify(notificationId, builder.build())
    }
    

保存传递到 NotificationManagerCompat.notify() 的通知 ID，因为如果之后想要更新或移除通知，将需要使用这个 ID。
注意：从 Android 8.1（API 级别 27）开始，应用每秒最多只能发出一次通知提示音。如果应用在一秒内发出了多条通知，这些通知都会按预期显示，但是每秒中只有第一条通知发出提示音。
添加操作按钮

一个通知最多可以提供三个操作按钮，让用户能够快速响应，例如暂停提醒，甚至回复短信。但这些操作按钮不应该重复用户在点按通知时执行的操作。

图 2. 带有一个操作按钮的通知

如需添加操作按钮，将 addAction() 传递给 PendingIntent 方法。这就像在设置通知的默认点按操作，不同的是不会启动 Activity，而是可以完成各种其他任务，例如启动在后台执行作业的 BroadcastReceiver，这样该操作就不会干扰已经打开的应用。

例如，以下代码演示了如何向特定接收者发送广播：
    val snoozeIntent = Intent(this, MyBroadcastReceiver::class.java).apply {
        action = ACTION_SNOOZE
        putExtra(EXTRA_NOTIFICATION_ID, 0)
    }
    val snoozePendingIntent: PendingIntent =
        PendingIntent.getBroadcast(this, 0, snoozeIntent, 0)
    val builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Hello World!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .addAction(R.drawable.ic_snooze, getString(R.string.snooze),
                    snoozePendingIntent)
    

如需详细了解如何构建 BroadcastReceiver 以运行后台工作，参阅广播指南。

如果您尝试构建包含媒体播放按钮（例如暂停和跳过曲目）的通知，参阅如何创建包含媒体控件的通知。
注意：在 Android 10（API 级别 29）和更高版本中，如果应用不提供自己的通知操作按钮，则平台会自动生成通知操作按钮。如果不希望应用通知显示任何建议的回复或操作，可以使用 setAllowGeneratedReplies() 和 setAllowSystemGeneratedContextualActions() 选择停用系统生成的回复和操作。
添加直接回复操作

Android 7.0（API 级别 24）中引入的直接回复操作允许用户直接在通知中输入文本，然后会直接提交给应用，而不必打开 Activity。例如，可以使用直接回复操作让用户从通知内回复短信或更新任务列表。

图 3. 点按“回复”按钮会打开文本输入框

直接回复操作在通知中显示为一个额外按钮，可打开文本输入。当用户完成输入后，系统会将文本回复附加到为通知操作指定的 Intent，然后将 Intent 发送到应用。
添加回复按钮

如需创建支持直接回复的通知操作，按照如下所述操作：

    创建一个可添加到通知操作的 RemoteInput.Builder 实例。此类的构造函数接受系统用作文本输入键的字符串。之后，手持式设备应用使用该键检索输入的文本。
    // Key for the string that's delivered in the action's intent.
    private val KEY_TEXT_REPLY = "key_text_reply"
    var replyLabel: String = resources.getString(R.string.reply_label)
    var remoteInput: RemoteInput = RemoteInput.Builder(KEY_TEXT_REPLY).run {
        setLabel(replyLabel)
        build()
    }
    

为回复操作创建 PendingIntent。
    // Build a PendingIntent for the reply action to trigger.
    var replyPendingIntent: PendingIntent =
        PendingIntent.getBroadcast(applicationContext,
            conversation.getConversationId(),
            getMessageReplyIntent(conversation.getConversationId()),
            PendingIntent.FLAG_UPDATE_CURRENT)
    

注意：如果重复使用 PendingIntent，则用户回复的会话可能不是他们所认为的会话。必须为每个会话提供一个不同的请求代码，或者提供对任何其他会话的回复 Intent 调用 equals() 时不会返回 true 的 Intent。会话 ID 会作为 Intent 的 extra 包的一部分被频繁传递，但调用 equals() 时会被忽略。
使用 addRemoteInput() 将 RemoteInput 对象附加到操作上。
    // Create the reply action and add the remote input.
    var action: NotificationCompat.Action =
        NotificationCompat.Action.Builder(R.drawable.ic_reply_icon,
            getString(R.string.label), replyPendingIntent)
            .addRemoteInput(remoteInput)
            .build()
    

对通知应用操作并发出通知。
        // Build the notification and add the action.
        val newMessageNotification = Notification.Builder(context, CHANNEL_ID)
                .setSmallIcon(R.drawable.ic_message)
                .setContentTitle(getString(R.string.title))
                .setContentText(getString(R.string.content))
                .addAction(action)
                .build()

        // Issue the notification.
        with(NotificationManagerCompat.from(this)) {
            notificationManager.notify(notificationId, newMessageNotification)
        }
        

当用户触发通知操作时，系统会提示用户输入回复，如图 3 所示。
从回复中检索用户输入

如需从通知回复界面接收用户输入，调用 RemoteInput.getResultsFromIntent() 并传入 BroadcastReceiver 收到的 Intent：
    private fun getMessageText(intent: Intent): CharSequence? {
        return RemoteInput.getResultsFromIntent(intent)?.getCharSequence(KEY_TEXT_REPLY)
    }
    

处理完文本后，必须使用相同的 ID 和标记（如果使用）调用 NotificationManagerCompat.notify() 来更新通知。若要隐藏直接回复界面并向用户确认他们的回复已收到并得到正确处理，则必须完成该操作。
    // Build a new notification, which informs the user that the system
    // handled their interaction with the previous notification.
    val repliedNotification = Notification.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_message)
            .setContentText(getString(R.string.replied))
            .build()

    // Issue the new notification.
    NotificationManagerCompat.from(this).apply {
        notificationManager.notify(notificationId, repliedNotification)
    }
    

在处理这个新通知时，使用传递给接收者的 onReceive() 方法的上下文。

还应通过调用 setRemoteInputHistory() 将回复附加到通知底部。但如果要构建即时通讯应用，应创建消息式通知，并在会话中附加新消息。





添加进度条

通知可以包含动画形式的进度指示器，向用户显示正在进行的操作的状态。
如果可以估算操作在任何时间点的完成进度，应通过调用 setProgress(max, progress, false) 使用指示器的“确定性”形式（如图 4 所示）。第一个参数是“完成”值（如 100）；第二个参数是当前完成的进度，最后一个参数表明这是一个确定性进度条。

随着操作的继续，持续使用 progress 的更新值调用 setProgress(max, progress, false) 并重新发出通知。
    val builder = NotificationCompat.Builder(this, CHANNEL_ID).apply {
        setContentTitle("Picture Download")
        setContentText("Download in progress")
        setSmallIcon(R.drawable.ic_notification)
        setPriority(NotificationCompat.PRIORITY_LOW
    }
    val PROGRESS_MAX = 100
    val PROGRESS_CURRENT = 0
    NotificationManagerCompat.from(this).apply {
        // Issue the initial notification with zero progress
        builder.setProgress(PROGRESS_MAX, PROGRESS_CURRENT, false)
        notify(notificationId, builder.build())

        // Do the job here that tracks the progress.
        // Usually, this should be in a
        // worker thread
        // To show progress, update PROGRESS_CURRENT and update the notification with:
        // builder.setProgress(PROGRESS_MAX, PROGRESS_CURRENT, false);
        // notificationManager.notify(notificationId, builder.build());

        // When done, update the notification one more time to remove the progress bar
        builder.setContentText("Download complete")
                .setProgress(0, 0, false)
        notify(notificationId, builder.build())
    }
    

操作结束时，progress 应该等于 max。可以在操作完成后仍显示进度条，也可以将其移除。无论哪种情况，都请记得更新通知文本，显示操作已完成。如需移除进度条，请调用 setProgress(0, 0, false)。
注意：由于进度条要求应用持续更新通知，因此该代码通常应在后台服务中运行。

如需显示不确定性进度条（不指示完成百分比的进度条），请调用 setProgress(0, 0, true)。结果会产生一个与上述进度条样式相同的指示器，区别是这个进度条是一个不指示完成情况的持续动画。在调用 setProgress(0, 0, false) 之前，进度动画会一直运行，调用后系统会更新通知以移除 Activity 指示器。

同时，请记得更改通知文本，表明操作已完成。
注意：如果实际需要下载一个文件，应考虑使用 DownloadManager，它会提供自己的通知来跟踪下载进度。
设置系统范围的类别

Android 使用一些预定义的系统范围类别来确定在用户启用勿扰模式后是否发出指定通知来干扰客户。

如果通知属于 NotificationCompat 中定义的预定义通知类别之一（例如 CATEGORY_ALARM、CATEGORY_REMINDER、CATEGORY_EVENT 或 CATEGORY_CALL），应通过将相应类别传递到 setCategory() 来进行声明。
    var builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Hello World!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setCategory(NotificationCompat.CATEGORY_MESSAGE)
    

系统使用这些关于通知类别的信息来决定当设备处于勿扰模式时是否显示通知。

但是，设置系统范围类别并不是必需操作，仅在通知符合 NotificationCompat 中定义的类别之一时才应这样做。
显示紧急消息

应用可能需要显示紧急的时效性消息，例如来电或响铃警报。在这些情况下，可以将全屏 Intent 与通知关联。调用通知时，根据设备的锁定状态，用户会看到以下情况之一：

    如果用户设备被锁定，会显示全屏 Activity，覆盖锁屏。
    如果用户设备处于解锁状态，通知以展开形式显示，其中包含用于处理或关闭通知的选项。

注意：包含全屏 Intent 的通知有很强的干扰性，因此这类通知只能用于最紧急的时效性消息。
注意：如果应用的目标平台是 Android 10（API 级别 29）或更高版本，必须在应用清单文件中请求 USE_FULL_SCREEN_INTENT 权限，以便系统启动与时效性通知关联的全屏 Activity。

以下代码段展示了如何将通知与全屏 Intent 关联：
    val fullScreenIntent = Intent(this, ImportantActivity::class.java)
    val fullScreenPendingIntent = PendingIntent.getActivity(this, 0,
        fullScreenIntent, PendingIntent.FLAG_UPDATE_CURRENT)

    var builder = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.notification_icon)
            .setContentTitle("My notification")
            .setContentText("Hello World!")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setFullScreenIntent(fullScreenPendingIntent, true)
   
   
   
   
   
设置锁定屏幕公开范围

如需控制锁定屏幕中通知的可见详情级别，请调用 setVisibility() 并指定以下值之一：

    VISIBILITY_PUBLIC 显示通知的完整内容。
    VISIBILITY_SECRET 不在锁定屏幕上显示该通知的任何部分。
    VISIBILITY_PRIVATE 显示基本信息，例如通知图标和内容标题，但隐藏通知的完整内容。

当设置 VISIBILITY_PRIVATE 时，还可以提供通知内容的备用版本，以隐藏特定详细信息。例如，短信应用可能会显示一条通知，提示“有 3 条新短信”，但是隐藏了短信内容和发件人。如需提供此备用通知，首先请像平时一样使用 NotificationCompat.Builder 创建备用通知。然后使用 setPublicVersion() 将备用通知附加到普通通知中。

但是，对于通知在锁定屏幕上是否可见，用户始终拥有最终控制权，甚至可以根据应用的通知渠道来控制公开范围。
更新通知

如需在发出此通知后对其进行更新，请再次调用 NotificationManagerCompat.notify()，并将之前使用的具有同一 ID 的通知传递给该方法。如果之前的通知已被关闭，则系统会创建一个新通知。

可以选择性调用 setOnlyAlertOnce()，这样通知只会在通知首次出现时打断用户（通过声音、振动或视觉提示），而之后更新则不会再打断用户。
注意：Android 会在更新通知时应用速率限制。如果过于频繁地发布对某条通知的更新（不到一秒内发布多条通知），系统可能会丢弃部分更新。
移除通知

除非发生以下情况之一，否则通知仍然可见：

    用户关闭通知。
    用户点击通知，且在创建通知时调用了 setAutoCancel()。
    针对特定的通知 ID 调用了 cancel()。此方法还会删除当前通知。
    调用了 cancelAll() 方法，该方法将移除之前发出的所有通知。
    如果在创建通知时使用 setTimeoutAfter() 设置了超时，系统会在指定持续时间过后取消通知。如果需要，可以在指定的超时持续时间过去之前取消通知。

有关即时通讯应用的最佳做法

使用下文所列的最佳做法作为快速参考，了解在为消息和聊天应用创建通知时应注意哪些要点。
使用 MessagingStyle

从 Android 7.0（API 级别 24）起，Android 提供了专用于消息内容的通知样式模板。使用 NotificationCompat.MessagingStyle 类，可以更改在通知中显示的多个标签，包括会话标题、其他消息和通知的内容视图。

以下代码段展示了如何使用 MessagingStyle 类自定义通知的样式。
    var notification = NotificationCompat.Builder(this, CHANNEL_ID)
            .setStyle(NotificationCompat.MessagingStyle("Me")
                    .setConversationTitle("Team lunch")
                    .addMessage("Hi", timestamp1, null) // Pass in null for user.
                    .addMessage("What's up?", timestamp2, "Coworker")
                    .addMessage("Not much", timestamp3, null)
                    .addMessage("How about lunch?", timestamp4, "Coworker"))
            .build()
    

从 Android 8.0（API 级别 26）起，使用 NotificationCompat.MessagingStyle 类的通知会在采用折叠形式时显示更多内容。还可以使用 addHistoricMessage() 方法，通过向与消息相关的通知添加历史消息为会话提供上下文。

使用 NotificationCompat.MessagingStyle 时：

    调用 MessagingStyle.setConversationTitle()，为超过两个用户的群聊设置标题。一个好的会话标题可能是群组名称，也可能是会话参与者的列表（如果没有具体名称）。否则，消息可能会被误以为属于与会话中最近消息发送者的一对一会话。
    使用 MessagingStyle.setData() 方法包含媒体消息，如图片等。目前支持 image/* 格式的 MIME 类型。
