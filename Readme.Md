# ICloud Drive

Để có thể truy cập vào `iCloud-Drive` với toàn quyền truy cập, ta add những dòng này vào file `info.plist`.

```swift
<key>CFBundleDocumentTypes</key>
	<array>
		<dict>
			<key>CFBundleDocumentTypes</key>
			<array>
				<dict>
					<key>CFBundleTypeIconFiles</key>
					<array/>
					<key>CFBundleTypeName</key>
					<string>OBJ model</string>
					<key>CFBundleTypeRole</key>
					<string>Editor</string>
					<key>LSHandlerRank</key>
					<string>Owner</string>
					<key>LSItemContentTypes</key>
					<array>
						<string>obj</string>
					</array>
				</dict>
			</array>
			<key>CFBundleTypeIconFiles</key>
			<array/>
			<key>CFBundleTypeName</key>
			<string>Obj file</string>
			<key>LSItemContentTypes</key>
			<array>
				<string>com.giena.Interface.document.obj</string>
			</array>
		</dict>
	</array>

	<key>UTExportedTypeDeclarations</key>
	<array>
		<dict>
			<key>UTTypeConformsTo</key>
			<array>
				<string>public.data</string>
			</array>
			<key>UTTypeDescription</key>
			<string>OBJ model</string>
			<key>UTTypeIconFiles</key>
			<array/>
			<key>UTTypeIdentifier</key>
			<string>com.giena.Interface.document.obj</string>
			<key>UTTypeTagSpecification</key>
			<dict>
				<key>public.filename-extension</key>
				<array>
					<string>obj</string>
				</array>
			</dict>
		</dict>
		<dict/>
	</array>
```

Để có thể access `icloud-Drive`, ta sẽ sử dụng `UIDocumentPickerViewController`:

```swift
var documentPicker: UIDocumentPickerViewController

if #available(iOS 14.0, *) {
    let supportedTypes: [UTType] = [.movie, .mpeg4Movie, .audio,. mp3, .video, .audiovisualContent]
    documentPicker = UIDocumentPickerViewController(forOpeningContentTypes: supportedTypes, asCopy: true)

} else {
    documentPicker = UIDocumentPickerViewController(documentTypes: [kUTTypeAudio as String, kUTTypeMP3 as String, kUTTypeMPEG4 as String, kUTTypeAVIMovie as String], in: UIDocumentPickerMode.import)
}
documentPicker.delegate = self
self.present(documentPicker, animated: true, completion: nil)
```

Ta chú ý về việc khởi tạo như sau:

```swift
UIDocumentPickerViewController(forOpeningContentTypes: supportedTypes, asCopy: true)
UIDocumentPickerViewController(documentTypes: [kUTTypeAudio as String, kUTTypeMP3 as String, kUTTypeMPEG4 as String, kUTTypeAVIMovie as String], in: UIDocumentPickerMode.import)
```

Hai biến `asCopy` và `in` sẽ thông báo cho hệ thống rằng: `data` nhận được sẽ chỉ có quyền read hay có cả quyền read và write. Như ví dụ trên, khi ta xét 2 para đó lần lượt là `true` và `imporrt`, thì dữ liệu có thể được copy sang thư mục khác(ở đây là ta load Data lên, rồi copy vào `fileManager`). Xét là `false` và `open` sẽ ngược lại

- Sau khi ta tích vào file, thì sẽ vào `delegate`:

```php
extension ImportController:  UIDocumentPickerDelegate, UINavigationControllerDelegate {
    func documentPicker(_ controller: UIDocumentPickerViewController, didPickDocumentsAt urls: [URL]) {
        guard let selectedFileURL = urls.first else {
            return
        }

        let filename = selectedFileURL.lastPathComponent
        
        guard let urlSave = URL.mediaFolder()?.appendingPathComponent(filename) else { return }
        if FileManager.default.fileExists(atPath: urlSave.path) {
            DispatchQueue.main.async {
                self.view.displayToast("Failed to export: File exists")
            }
            return
        }
        
        let loadingView = LoadingAnimationView()
        loadingView.setMessage("Exporting video...")
        loadingView.show()
        
        do {
            try FileManager.default.copyItem(at: selectedFileURL, to: urlSave)
            self.saveMusicToDB(url: urlSave, musicSourceType: .others, filename: filename)
            DispatchQueue.main.async {
                self.view.displayToast("Export video successfully")
            }
        } catch (let err) {
            print("DEBUG: caccaca \(err.localizedDescription)")
        }
        
        DispatchQueue.main.async { loadingView.dismiss() }
    }

}
```