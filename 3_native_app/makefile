# Set the default parameters for the build, can be overridden with either an
# environment variable or by using `make target KEY=VALUE`.
VERSION ?= $(shell cat package.json | jq --raw-output .version)
BUILD_NUM ?= 0

debug:
	echo "version $(VERSION)"

# Touch will ensure that all version numbers are consistent, based on the
# package.json version.
touch:
  # Set the version and build number in the config file.
	sed -i "" -e 's/android-versionCode=\"[0-9]*\"/android-versionCode=\"$(BUILD_NUM)\"/g' ./config.xml
	sed -i "" -e 's/ios-CFBundleVersion=\"[0-9]*\"/ios-CFBundleVersion=\"$(BUILD_NUM)\"/g' ./config.xml
	sed -i "" -e 's/version=\"[.0-9a-zA-Z]*\"/version=\"$(VERSION)"/g' ./config.xml

# Tests the code.
test:
	$(warning No tests so far, do not forget to add some in the future.)
	@echo 0

# Creates the Android APK.
build-android:
ifndef IA_KEYSTORE_PASSWORD
	$(error No IA_KEYSTORE_PASSWORD is set, cannot build APK.)
endif
	cd android && ./gradlew assembleRelease && cd ..
	cp -f ./android/app/build/outputs/apk/*.apk ./artifacts

deploy-android: bump build-js build-android
	# Push the build to TestFairy.
	curl https://app.testfairy.com/api/upload \
		-F api_key='$(TESTFAIRY_API_KEY)' \
		-F "file=@artifacts/ionic_app.apk" \
		-F auto-update='on' \
		-F metrics='cpu,network,logcat' \
		-F options='shake'


# Creates the iOS IPA.
build-ios:
ifndef IA_CODE_SIGNING_IDENTITY
	$(error No IA_CODE_SIGNING_IDENTITY is set, cannot build IPA.)
endif
	# Recreate the iOS platform.
	rm -rf ./platforms/ios
	cordova platform add ios
	npm run ionic:build
	cordova prepare ios --release
	cd ./platforms/ios; fastlane gym --scheme "ionic_app" \
		--use_legacy_build_api \
		--codesigning_identity "$(RNA_CODE_SIGNING_IDENTITY)"; cd ../..;