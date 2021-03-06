# FirebaseChat
This project follows the tutorial at [Lets build that app](https://www.youtube.com/channel/UCuP2vJ6kRutQBfRmdcI92mA) of Brain Voong. Build an chat app with the Firebase 3 backend. It's a really useful tutorial for experienced iOS developers. 

In the tutorial, Brian used Swift 2.3, I started this on October, so that I used Swift 3. 

The greatest thing I learnt from Brian is he just uses Auto layout programmatically. Incredible. 

[Ep1](https://www.youtube.com/watch?v=lWSc0wHFTXM&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=1):  Build the login controller with Constraint Anchors. You can take a lot at my Youtube clone (followed another tutorial) to see my notes. There are some useful note here. Something is extend UIColor init, start app programmatically. 

[Ep2](https://www.youtube.com/watch?v=guFW9aj4EHM&index=2&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): Create app in Firebase console. Connect the project with the Firebase via SDK installed by Cocodpod. First data added to Firebase databse.

Create registered user to Firebase Authentication. 

	FIRAuth.auth()?.createUser(withEmail: email, password: password, completion: { (user, error) in
		// do your job here. 
	}
	
Save user information to Firebase Database. Firebase Database is a NoSQL, it uses node instead of table. 

	let ref = FIRDatabase.database().reference(fromURL: "https://your_app_url.firebaseio.com/")
    let node = ref.child("fatherItem").child("littleChildItem")
    let dataToSave = [
        "key": value,
    ]
    node.updateChildValues(values, withCompletionBlock: { (error, ref) in
		// do your job here
    })
    
[Ep3](https://www.youtube.com/watch?v=4rNtIeC_dsQ&index=3&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): Handle login function. Change the UI to toggle login and register. Login to the registered account with Firebase. 

	FIRAuth.auth()?.signIn(withEmail: email, password: password, completion: { (user, error) in 
		// do your job here
	}

[Ep4](https://www.youtube.com/watch?v=qD582zfXlgo&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=4): Fetch all users from Firebase. 

The observe type .childAdded will create a connection to Firebase and be called every time new user registered. 

	FIRDatabase.database().reference().child("users").observe(.childAdded, with: { (snapshot) in

		// your job goes here 
	}

[Ep5](https://www.youtube.com/watch?v=b1vrjt7Nvb0&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=5): Upload user images to Firebase Storage. 

The UIImagePickerController returns some types of images, remember 2 of them: `UIImagePickerControllerEditedImage` and `UIImagePickerControllerOriginalImage`

	if let editedImage = info["UIImagePickerControllerEditedImage"] as? UIImage {
        selectedImage = editedImage
    }
    else if let originalImage = info["UIImagePickerControllerOriginalImage"] as? UIImage  {
        selectedImage = originalImage
    }

Something Brian didn't mention to, I found it when I worked. 

- Permission to access photo gallery: You will have a crash with no reason, maybe it happens only in Swift 3. Let's add some lines into `Info.plist` file. 

		<key>NSPhotoLibraryUsageDescription</key>
		<string>This app requires access to the photo library.</string>
		<key>NSCameraUsageDescription</key>
		<string>This app requires access to the camera.</string>

- Permission to upload photo to Firebase: By default, Firebase doesn't allow to access to Storage or Database without authentication. You can change that in `Storage\Rule` or `Database\Rule`. It's useful when your app doesn't require register or login. 

Change this 

	allow read, write: if request.auth != null; 
	
to this 

	allow read, write: if request.auth == null;

I did some difference with Brian. Move the upload code to a function with a callback instead of upload inside the handler function. 

	uploadProfileImage(name: fileName, completionHandler: { (metadata) in
		// prepare data to save into databse
		// save data to databse 
	}

I use the Firebase user uid instead of generate new one to store to storage. 

	let fileName = uid + ".png"

[Ep6](https://www.youtube.com/watch?v=GX4mcOOUrWQ&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=6): Load images from Firebase and caching. The image from Firebase Storage has a url, stored to Firebase database. It's easy to download with 3rd lib such as Kingfisher or SDWebImage. They automatically cache the images. In this tutorial, Brian used URLSession to download and cache image manually. 

I learnt this in his building Youtube serial. So that I move code to my own cache class. Check it out at `ImageCaching.swift` file. Easily use with 

	avatarCaching.getImage(with: urlString) { (downloadedImage) in
		// update your image view
    }

[Ep7](https://www.youtube.com/watch?v=69LooiLYjQo&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=7): Use `UIImageJPEGRepresentation` instead of `UIImagePNGRepresentation` to reduce the image quality to upload and download faster. 

	let uploadData = UIImageJPEGRepresentation(image, 0.1)

Update the title bar with user profile image and name. I did differently with Brian, created a protocol and pass data from `LoginController` to `MessageController` instead of pass `MessageController` instance. 

[Ep8](https://www.youtube.com/watch?v=FDay6ocBlnE): Create chat log controller to show the conversation. Save messages to Firebase database at node `childByAutoId`

	let ref = FIRDatabase.database().reference().child("messages")
	let childRef = ref.childByAutoId()
	let values = ["text": inputTextField.text!]
	childRef.updateChildValues(values)
	
[Ep9](https://www.youtube.com/watch?v=cw0gLZHJOiE): Get all messages from database and show in the tableView. Shouldn't use Xcode autocorrect to finish Firebase function. It's better type closures yourself.
	
	let ref = FIRDatabase.database().reference().child("messages")
	ref.observe(.childAdded, with: { snapshot in
		// parse messages data and render to UI
	})

[Ep10](https://www.youtube.com/watch?v=fyqksNlC8ks): Easy way to group the message as user. Add all messages into a dictionary as the value with the user id is the key. 

	self.messagesDictionary[toId] = message

Convert time interval (double value) to Date and format date to string

	let timestamp = Date(timeIntervalSince1970: seconds)
    let dateFormatter = DateFormatter()
    dateFormatter.dateFormat = "hh:mm:ss a"
    timeLabel.text = dateFormatter.string(from: timestamp)

[Ep11](https://www.youtube.com/watch?v=K1AgGLoT54M&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=11&t=863s): create user_messages to to separate others user messages. From there, we can retrieve the messages content for the conversations. 

[Ep12](https://www.youtube.com/watch?v=azFjJZxZP6M&t=2s&index=12&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq#t=1679.170654): Load the user chat to the chat log controller. Filter the user message related to the current user and the selected user. 

[Ep13](https://www.youtube.com/watch?v=yhGw5bR46AQ&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=13): Calculate the text size to make the collection view cell fit to the text. 

	 private func estimateFrameForText(text: String) -> CGRect {
	    let size = CGSize(width: 200, height: 100)
	    let options = NSStringDrawingOptions.usesFontLeading.union(.usesLineFragmentOrigin)
	    return NSString(string: text).boundingRect(with: size, options: options, attributes: [NSFontAttributeName: UIFont.systemFont(ofSize: 15)], context: nil)
	}

Should add some pixels for height padding 

	height = estimateFrameForText(text: text).height + 20
	
[Ep14](https://www.youtube.com/watch?v=JK7pHuSfLyA&t=490s): Create a gray bubble chat for incoming messages. Show the friend profile in chat messages. 

Turn on and off the constraint to show the bubble view left or right 

	bubbleViewRightAnchor = bubbleView.rightAnchor.constraint(equalTo: rightAnchor, constant: -8)
	bubbleViewRightAnchor!.isActive = true
	bubbleViewLeftAnchor = bubbleView.leftAnchor.constraint(equalTo:profileImageView.rightAnchor, constant: 8)
	
	// turn on the left constraint, turn off the right constraint
	cell.bubbleViewLeftAnchor?.isActive = true
    cell.bubbleViewRightAnchor?.isActive = false

When observe an event from Firebase, the completion is called many times. So that the update UI codes will run a lot. A trick to cancel the previous code and just run the last time called. 

	var timer: Timer?
	self.timer?.invalidate()
	self.timer = Timer.scheduledTimer(timeInterval: 1, target: self, selector: #selector(self.handleReloadTable), userInfo: nil, repeats: false)
	
Another solution, don't want to create a timer. 

	NSObject.cancelPreviousPerformRequests(withTarget: self)
	perform(#selector(self.handleReloadTable), with: nil, afterDelay: 1)

[Ep15](https://www.youtube.com/watch?v=ky7YRh01by8): A great tip to show and hide keyboard. 

Override the inputAccessaryView to show the input box you want. 

	override var inputAccessoryView: UIView? {
	    get {
	        return inputContainerView
	    }
	}

It's still hidden from the screen. Override canBecomeFirstResponder always true 

	override var canBecomeFirstResponder: Bool {
        return true
    }
    
Hide keyboard when the collection view scroll down

	collectionView?.keyboardDismissMode = .interactive
	
The trandition way to show hide keyboard is NotificationCenter 

	func handleKeyboardWillHide(notification: Notification) {
        containerViewBottomAnchor?.constant = 0
        
        let keyboardDuration = (notification.userInfo![UIKeyboardAnimationDurationUserInfoKey] as! NSNumber).doubleValue
        
        UIView.animate(withDuration: keyboardDuration, animations: {
            
            self.view.layoutIfNeeded()
        })
    }
    
    func handleKeyboardWillShow(notification: Notification) {
        
        let keyboardFrame = (notification.userInfo![UIKeyboardFrameBeginUserInfoKey] as! NSValue).cgRectValue
        containerViewBottomAnchor?.constant = -keyboardFrame.height
        
        let keyboardDuration = (notification.userInfo![UIKeyboardAnimationDurationUserInfoKey] as! NSNumber).doubleValue
        
        UIView.animate(withDuration: keyboardDuration, animations: {
            
            self.view.layoutIfNeeded()
        })
    }

Don't forget register when key view controller didAppear

    NotificationCenter.default.addObserver(self, selector: #selector(handleKeyboardWillShow), name: NSNotification.Name.UIKeyboardWillShow, object: nil)
    
    NotificationCenter.default.addObserver(self, selector: #selector(handleKeyboardWillHide), name: NSNotification.Name.UIKeyboardWillHide, object: nil)

And unregister when didDisappear 

	NotificationCenter.default.removeObserver(self)
	
[Ep16](https://www.youtube.com/watch?v=8cN-jZcbTjg): A big trick to structure the Firebase database. The user messages have to arrange to structure user-messages\fromId\toId. So that, we only have to monitor that node, instead of monitor the all user-messages\fromId and filter toId to display. 

A small change in some places in code. Please follow the Ep16. 	

[Ep17](https://www.youtube.com/watch?v=R07TcmTR3w0&t=15s&index=17&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): A cool message with image. Select an image to send, upload to Firebase Storage and download to the message chat log. You can see how to upload to Firebase Storage in Ep2 note. 

[Ep18](https://www.youtube.com/watch?v=FqDVKW9Rn_M&t=15s&index=17#t=1705.002882): Add image width and height to database data so that we can easily calculate the image size for the message. 

A cool thing in the `sendMessageWithProperties` function is 

	properties.forEach({ values[$0] = $1 })
	
It's very easy to add new value to the `values` dictionary. Love this way. 

[Ep19](https://www.youtube.com/watch?v=fo3nSRBWfRA): A cool animation to view the image in full screen. Create a fake image view and zoom it in. We have to convert the real image view frame to the frame in the global view. 

	let startingFrame = startingImageView.superview?.convert(startingImageView.frame, to: nil)

Zoom the fake image view to frame `CGRect(x: 0, y: 0, width: keyWindow.frame.width, height: keyWindow.frame.height)`

[Ep20](https://www.youtube.com/watch?v=eRkpdRDYGeM): Select video from the `UIImagePickerController`, upload to Firebase and get the thumbnail to show in chat messsage log. 

Set mediaTypes to enable select video from `UIImagePickerController`. MobileCoreServices have to be imported before. 

	imagePicker.mediaTypes = [kUTTypeImage as String, kUTTypeMovie as String] 
	
Get the video URL to upload to Firebase 

	let videoUrl = info[UIImagePickerControllerMediaURL] as? URL
	let uploadTask = FIRStorage.storage().reference().child("message_movies").child(filename).putFile(url, metadata: nil, completion: { (metadata, error) in
	
	// generate thumbnail 
	// save thumbnail, thumbnail size and uploaded video url to the database
	
	}
	
	// handle to uploadTask progress. 

Get thumbnail from the local video file. Don't forget import AVFoundation

	private func thumbnailImageForFileUrl(fileUrl: URL) -> UIImage? {

        let asset = AVAsset(url: fileUrl)
        let imageGenerator = AVAssetImageGenerator(asset: asset)
        
        do {
            let time = CMTime(value: 1, timescale: 60)
            let thumbnailCGImage = try imageGenerator.copyCGImage(at: time, actualTime: nil)
            return UIImage(cgImage: thumbnailCGImage)
        }
        catch let err {
            print(err)
        }
        
        return nil
    }
  
Handle upload progress

	uploadTask.observe(.progress, handler: { snapshot in
        
        if let completedUnitCount = snapshot.progress?.completedUnitCount {
                self.navigationItem.title = String(completedUnitCount)
        }
        })
        
        uploadTask.observe(.success, handler: { snapshot in
         
            self.navigationItem.title = self.user?.name
        })

[Ep21](https://www.youtube.com/watch?v=4ISMTG-E3Po&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq&index=21): Play the video inside the chat log. 
	
	player = AVPlayer(url: url)
	playerLayer = AVPlayerLayer(player: player)
	playerLayer?.frame = bubbleView.bounds
	bubbleView.layer.addSublayer(playerLayer!)
	player?.play()
	
Instead of prevent tap on the video thumbnail, I stretch the button to fit the bubble view size. A little different from Brain. k
    
[Ep22](https://www.youtube.com/watch?v=KkHEEhftUk0&index=22&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): Remove messages from the message controller. We have to do 2 things.

Remove the mesasge from the message data. 

	override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
     
	    guard let uid = FIRAuth.auth()?.currentUser?.uid else { return }
	    
	    if let chatPartnerid = messages[indexPath.row].chatParnterId() {
	        FIRDatabase.database().reference().child("user-messages").child(uid).child(chatPartnerid).removeValue(completionBlock: { (error, ref) in
	            
	            if error != nil {
	                print("fail to delete message")
	            }
	
	            self.messagesDictionary.removeValue(forKey: chatPartnerid)
	            self.attemptReloadTable()
	        })
	    }
    }
   
Observe child remove from database so that when the messages are removed, the application updated. 

	ref.observe(.childRemoved, with: { snapshot in
         
        self.messagesDictionary.removeValue(forKey: snapshot.key)
        self.attemptReloadTable()
    })
    
[Ep23](https://www.youtube.com/watch?v=F3snOdQ5Qyo&index=23&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): Refactor code to move input container view out of the controller. Now we can reuse this view anywhere we want. 

[Ep24](https://www.youtube.com/watch?v=ICEB51UE6sU&index=24&list=PL0dzCUj1L5JEfHqwjBV0XFb9qx9cGXwkq): Convert Swift 2.2 to Swift 3. But my project started with Swift 3. So that, I did nothing here :D 


-
This is the end of the development from Brain. I will be back to finish it later. 
