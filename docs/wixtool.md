# 설치

* 다음의 URL에서 최신 버전의 WiX를 받아 설치를 진행 한다.

    [WIX TOOL SET](http://wixtoolset.org/releases/)

---
<br/>
# 개요

 WiX toolset은 xml을 이용하여 윈도우 설치 파일을 생성해 낸다.  
 UI를 지원하는 기본적인 설치 프로젝트를 사용하여 설치 파일을 생성하는 것과는  
 달리 프로그래밍 언어로 코딩을 하는 것처럼 설치 파일을 생성하게 된다. 

 WiX toolset을 사용하게 될 경우의 이점은 다음과 같다.

* 선언적인 접근.(declarative approach)
* 제한 없이 윈도우 설치 기능들에 접근 할 수 있다.(unrestricted access to Windows Installer fuctionality)
* GUI 기반이 아닌 Source code 기반이다,(? source code instead of GUI-based assembly of information)
* 어플리케이션 빌드 프로세스를 완벽하게 통합 할 수 있다.(complete integration into application build process)
* 어플리케이션 개발과 통합이 가능 하다.(possible integration with application development)
* 팀 단위 개발을 지원한다.(support for team development, both in-house and third-party)
* 무료다.(free, opensource)

---
<br/>
# 정말 간단한 설치 파일 생성 예제

* WIX를 이용하여 간단한 설치 파일을 생성해 보자!

  **Step 1 : C# 윈도우 어플리케이션 프로젝트를 생성 한다.**

    1. C# 윈도우 어플리케이션 프로젝트 생성.
    1. 셍략.

  **Step 2 : 인스톨 프로젝트를 생성 한다.**

    1. 솔루션내 새 프로젝트 생성.
    1. Windows Installer XML 프로젝트 선택.
    1. C# 윈도우 어플리케이션 프로젝트를 설치 프로젝트에 참조 추가.
    1. 설치 프로젝트 내 Product.wxs 파일을 확인.	
    1. <Product> 항목의 Manufacturer="" 필드에 게시자를 작성해 준다. ex)Manufacturer="SHSystem"
    1. <Component> 주변의 주석을 해제한 뒤 TODO의 내용을 삭제 한다.
    1. <Component> 내 다음을 삽입 <File Source="$(var.MyApplication.TargetPath)" />
		* MyApplication에는 생성한 C# 윈도우 어플리케이션 프로젝트 명칭을 써준다.
    1. 설치 프로젝트 빌드 후 C:\Program Files (x86)\[Produect Name]에 설치가 제대로 진행 되었는지 확인 한다.

```
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
	<Product Id="*" Name="SetupProject1" Language="1033" Version="1.0.0.0" Manufacturer="" UpgradeCode="12a0089a-c561-4bca-ac26-d9f9b244d753">
		<Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />

		<MajorUpgrade DowngradeErrorMessage="A newer version of [ProductName] is already installed." />
		<MediaTemplate />

		<Feature Id="ProductFeature" Title="SetupProject1" Level="1">
			<ComponentGroupRef Id="ProductComponents" />
		</Feature>
	</Product>

	<Fragment>
		<Directory Id="TARGETDIR" Name="SourceDir">
			<Directory Id="ProgramFilesFolder">
				<Directory Id="INSTALLFOLDER" Name="SetupProject1" />
			</Directory>
		</Directory>
	</Fragment>

	<Fragment>
		<ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
			<!-- TODO: Remove the comments around this Component element and the ComponentRef below in order to add resources to this installer. -->
			<!-- <Component Id="ProductComponent"> -->
				<!-- TODO: Insert files, registry keys, and other resources here. -->
			<!-- </Component> -->
		</ComponentGroup>
	</Fragment>
</Wix>
```

---

<br/>
# 프로젝트 파일 살펴 보기

Window Installer에 익숙하다면 .wxs 파일의 구조를 쉽게 파악 할 수 있다.

첫째로, WiX element는 파일안의 내용을 감싸기 위해 존재 한다.

두번째로, Product element는 Windows Installer가 제품을 확인하기 위해 필요한 속성([ProductCode](https://msdn.microsoft.com/library/Aa370854.aspx), [ProductName](https://msdn.microsoft.com/library/Aa370857.aspx), [ProductLanguage](https://msdn.microsoft.com/library/Aa370856.aspx), [ProductVersion](https://msdn.microsoft.com/library/Aa370859.aspx))들을 정의한다.

세번째로, Package element는 설치 패키지 자체의 [정보](https://msdn.microsoft.com/library/Aa372045.aspx)를 제공하는 속성들을 담고 있다.

ComponentRef element를 제외한 나머지 element들은 Windows Installer tables과 이름으로 맵핑된다.
(ex [Directory Table](https://msdn.microsoft.com/library/Aa368295.aspx), [Component table](https://msdn.microsoft.com/library/Aa368295.aspx), [Feature table](https://msdn.microsoft.com/library/Aa368585.aspx))

The Component element is tied to the Features which maps to the entries in the [FeatureComponents table](https://msdn.microsoft.com/library/Aa368579.aspx).

---

<br/>
# [Visual Stuio 연동](http://wixtoolset.org/documentation/manual/v3/votive/)


<h3>Project Template</h3>

 WiX Visual Studio 패키지는 다음과 같은 Project Template을 제공 한다.

  **1. WiX Project :** 새로운 윈도우 설치 패키지(.mis)를 생성 하는 프로젝트. 프로젝트는 .wxs 파일을 포함한다. .wxs 파일은 Product라는 속성으로 구성되어 있으며, 해당 속성은 WiX가 윈도우 설치 패키지를 생성 할 수 있도록 권한을 부여하는 최소한의 정보를 가지고 있다. Product는 Package, Media, Directory, Component, Feature등의 속성을 포함 하고 있다.

  **2. WiX Library Project :** WiX Library 프로젝트를 생성하는 프로젝트.

  **3. WiX Merge Module Project :** Windows Installer Merge module(.msm)을 생성하는 프로젝트.

<h3>Item Template</h3>
  WiX Visual Stuido 패키지에서는 다음의 파일 템플릿을 제공 한다.

  **1. WiX File :** a .wxs file pre-populated with the same information as the default WXS file in a WiX Library Project

  **2. WiX Include File :** a blank .wxi file

  **3. WiX Localization File :**  a blank .wxl file

  **4. Text File :**  a blank .txt file

  자세한 사항은 다음의 [파일 타입](http://wixtoolset.org/documentation/manual/v3/overview/files.html)을 참고.
