---
layout: post
title: "안드로이드 SQLiteOpenHelper의 onCreate와 onUpgrade"
description: "안드로이드 SQLiteOpenHelper의 onCreate와 onUpgrade"
category: Android
tags: [Android]
author: FreeBz
---
Head First Android Delopment 를 보고, 처음으로 안드로이드 앱을 작성해 보았다.

10여차례 기능을 추가하여 버전업 하는 동안 큰 문제가 없었는데, 16번째 판올림에서 버그가 발생했다.


Head First Androd 에서는 DB 버전관리에 관련해서, 다음과 같은 가이드 하고 있다.

	public void onCreate(SQLiteDatabase database) {
		database.execSQL("CREATE TABLE ....");
	}

	public void onUpgrade(SQLiteDatabase database, int oldVersion, int newVersion) {
		database.execSQL("DROP TABLE ...");
		onCreate(database);
	}


위와 같이 onUpgrade에서 테이블을 삭제하고, onCreate 메서드를 호출해서 테이블을 재 생성해주는 것은 Database 버전을 올릴 때마다 유저의 데이터 정보를 분실하게 됨을 뜻한다.

따라서 onUpgrade 메서드에서는 위처럼 DB를 초기화 하는 것이 아닌 DB의 스키마 변경 내용을 반영해 주어야 한다.

그런데 DB 스키마의 변경 내용은 onCreate에도 동일하게 적용해야 한다. DB버전이 올라간 뒤에서 신규로 설치하는 유저가 있기 때문이다.


나는 처음에 DB스키마 관련 내용을 private 메서드로 만들어서
onCreate와 onUpgrade 메서드에 동일하게 적용하였다.

또한 onUpgrade 메서드에서는 oldVersion을 비교하여, 아주 오래전 버전에서 업데이트 하는 유저의 경우, 필요한 스키마가 순서대로 적용되도록 구현하였다.


내가 만든 실수는 새로운 스키마를 적용하는데 onUpgrade 메서드에만 구현을 추가한 점이었다. 스키마 관련 메서드도 내용이 늘어나다 보니, 코드가 길어져서, 그만 onCreate에도 동일하게 추가해야 하는 것을 잊고 말았다.

같은 실수를 되풀이 하고 싶지 않아 적용한 방법은 다음과 같다.

	public void onCreate(SQLiteDatabase database) {
		onUpgrade(database, -1, 1);
	}

	public void onUpgrade(SQLiteDatabase database, int oldVersion, int newVersion) {
		if (oldVersion < 0) {
			// 앱이 처음 설치 될때 적용할 내용
		}

		if (oldVersion < 30) {
			// DB버전이 30 이전까지 스키마 변경 내용
		}

		if (oldVersion < 40) {
			// DB버전이 40 이전까지 스키마 변경 내용
		}
	}

이와 같이 onUpgrade메서드에서 모든 변경이력을 관리하는 편이 코드 중복도 없고, 실수를 방지하기에도 좋은 것 같다.

