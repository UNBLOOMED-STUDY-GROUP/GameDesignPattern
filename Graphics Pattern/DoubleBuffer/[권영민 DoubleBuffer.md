
### 의도

- 여러 순차 작업의 결과를 한 번에 보여준다.

### 동기

- 순차적으로 혹은 동시에 진행되는 여러 작업을 한번에 모아서 봐야 할 때
- 게임의 렌더링
    - 화면을 그리는 중간 과정을 보이는 게 아니라 장면은 부드럽고 빠르게 업데이트 되어야 하고 매 프레임 **완성되면 한 번에 보여줘야 한다**.</br>

## 컴퓨터 그래픽스 작동 원리

### 렌더링 방식

- 한 번에 한 픽셀을 그린다.
- 화면 왼쪽에서 오른쪽으로 한 줄 그린 후 다음 줄로 내려간다.
- 화면 우하단에 도달하면 다시 좌상단으로 돌아가서 같은 작업을 반복한다.

### 어떤 색을 어디에 칠할까 : 프레임버퍼

- 대부분의 컴퓨터에서는 픽셀을 `프레임버퍼` 로부터 가져온다.
- `프레임버퍼` 로부터 한 바이트씩 색깔 값을 읽어온다.
- 프레임 버퍼
    - 메모리에 할당된 펙셀들의 배열로, 한 픽셀의 색을 여러 바이트로 표현하는 RAM의 한 부분이다.

### 문제 상황 : 테어링(화면 찢김 현상)
![image](https://github.com/user-attachments/assets/094db492-45d6-43b5-8726-040a97f80e06)
- 우리가 입력해 놓은 픽셀 값을 **비디오 드라이버**가 화면에 출력하면서 게임 화면이 나오기 시작하지만 **아직 다 입력하지 못한 버퍼 값까지 화면에 출력** 될 수 있다.
- 비주얼 버그가 발생한다. (테어링)

### 해결 방법 → 이중 버퍼 패턴
- 코드에서는 픽셀을 한 번에 하나씩 그리되, 비디오 드라이버는 전체 픽셀을 한 번에 다 읽을 수 있어야 한다.
- 즉, 이전 프레임에는 이미지가 하나도 안보이다가, 다음 프레임에 전체가 보여야 한다.

### 이중 버퍼 패턴
![image](https://github.com/user-attachments/assets/2500e923-6b7e-4646-9013-8ecfeb2d220d)
- 프레임 버퍼를 **두 개** 준비한다.
- 하나의 버퍼(버퍼 A)에는 지금 프레임에 보여지는 데이터를 둬서 GPU가 원할 때, 언제든지 읽을 수 있게 한다.
- 그동안 렌더링 코드는 다른 프레임 버퍼(버퍼 B)를 채운다.
- 렌더링 코드가 장면을 다 그린 후에는 조명을 바꾸는 것처럼 버퍼를 교체한 뒤에 비디오 하드웨어에 지금부터는 버퍼 B를 읽으라고 알려준다.
- 화면 깜빡임에 맞춰 버퍼가 바뀌기 때문에 테어링은 더 이상 생기지 않고 전체 장면이 한번에 나타나게 된다.
- 그 사이에 교체된 이전 프레임 버퍼에 다음 프레임에 들어갈 화면을 그리면 된다.</br>


## 패턴

### 원리

- 버퍼 클래스는 변경이 가능한 상태인 버퍼를 **캡슐화** 한다.
- 버퍼는 점차적으로 수정되지만, 밖에서는 한 번에 바뀌는 것처럼 보이게 하고 싶다.
- 이를 위해 버퍼 클래스는 현재 버퍼(정보를 읽을 때)와 다음 버퍼(정보를 쓸 때) 두 개의 버퍼를 갖는다.
- 변경이 끝나면 다음 버퍼와 현재 버퍼를 교체해  다음 버퍼가 보여지게 한다.
- 현재 버퍼는 새로운 다음 버퍼가 되어 재 사용된다.

### 언제 쓸 것인가

- 순차적으로 변경해야 하는 상태가 있다.
- 이 상태는 변경 도중에도 접근 가능해야 한다.
- 바깥 코드에서는 작업 중인 상태에 접근할 수 없어야 한다.
- 상태에 값을 쓰는 도중에도 기다리지 않고 바로 접근할 수 있어야 한다.

### 주의 사항

- 교체 연산 자체에 시간이 걸린다.
    - 버퍼에 값을 쓰는 것보다 교체가 더 오래 걸린다면 이중버퍼 패턴은 아무런 도움이 안된다.
- 버퍼가 두 개 필요하다.
    - 메모리 버퍼 두 곳에 항상 쌍으로 가지고 있어야 하기 때문에 메모리가 부족한 기기에서는 굉장히 부담이 될 수 있다.</br>
## 예제 1 : 그래픽스

### 버퍼

```cpp
class Framebuffer 
{
public:
	Framebuffer() { Clear(); }
	void Clear() 
	{
		for (int i = 0; i < WIDTH * HEIGHT; ++i) 
		{
			pixels[i] = WHITE;
		}
	}
	void Draw(int x, int y) 
	{
		pixels[(WIDTH * y) + x] = BLACK;
	}
	const char* GetPixels() { return pixels; }
private:
	static const int WIDTH = 160;
	static const int HEIGHT = 120;

	char pixels[WIDTH*HEIGHT];
};
```

- `Clear()` : 전체 버퍼를 흰색으로 채운다.
- `Draw()` : 특정 픽셀에 검은색을 입력한다.
- `GetPixels()` : 픽셀 데이터를 담고 있는 메모리 배열에 접근 가능하다.

### 씬

```cpp
class Scene 
{
public:
	void Draw() 
	{
		Buffer.Clear();
		Buffer.Draw(1, 1); Buffer.Draw(4, 1);
		Buffer.Draw(1, 3); Buffer.Draw(2, 4);
		Buffer.Draw(3, 4); Buffer.Draw(4, 3);
	}
	Framebuffer& GetBuffer() { return Buffer; }
private:
	Framebuffer Buffer;
};
```

- Scene 클래스 안에서 여러번 `Draw()` 를 호출해 버퍼에 권하는 그림을 그린다.
- 먼저 버퍼를 지운 뒤 한 번에 하나 씩 그리고자 하는 픽셀을 찍는다.
- `GetBuffer()` : 동시에 `비디오 드라이버`에서 **내부 버퍼에 접근**할 수 있도록 `GetBuffer()` 를 제공한다.

### 문제 상황 : 비디오 드라이버가 작업중인 버퍼에 접근할 수 있다.

- 비디오 드라이버가 아무때나 `GetPixel()` 함수를 호출해 버퍼에 접근할 수 있기 때문에 문제가 발생한다.
    - 렌더링하는 도중 어딘가에서 비디오 드라이버가 버퍼를 읽어버릴 수 있다.

```cpp
		buffer.Draw(1, 1); buffer.Draw(4, 1);
		// 이때 비디오 드라이버가 픽셀 퍼버 전체를 읽을 수도 있다.
		buffer.Draw(1, 3); buffer.Draw(2, 4);
		buffer.Draw(3, 4); buffer.Draw(4, 3);
```

### 해결 : 이중 버퍼

```cpp
class Scene 
{
public:
	Scene() : Current(&Buffers[0]), Next(&Buffers[1]) {  }
	void Draw() 
	{
		Next->Clear();
		Next->Draw(1, 1);
		// ... 
		Next->Draw(4, 3);
		Swap();
	}
	Framebuffer& GetBuffer() { return *Current; }

private:
	void Swap() 
	{
	// 버퍼 포인터만 교체한다.
		Framebuffer* temp = Current;
		Current = Next;
		Next = temp;
	}

	Framebuffer Buffers[2];
	Framebuffer* Current;
	Framebuffer* Next;
};
```

- 버퍼 두개가 배열에 들어있다.
- 버퍼에 접근할 때는 배열 대신 Next와 Current 포인터 멤버 변수로 접근한다.
- 렌더링할 때는 Next 포인터가 가리키는 다음 버퍼에 그리고, 비디오 드라이버는 Current 포인터로 현재 버퍼에 접근해 픽셀을 가져온다.
- 비디오 드라이버가 작업 중인 버퍼에 접근하는 걸 막을 수 있다.
- 장면을 다 그린 후에 `Swap()` 을 호출한다.
- `Swap()` 함수에서는 `Next` 와  `Current` 포인터를 바꾼다.
- 이제 비디오 드라이버가 `GetBuffer()` 를 호출하면 이전에 화면에 그리기 위해 사용한 버퍼 대신 방금 그린 화면이 들어 있는 버퍼를 얻게 된다.</br>

### Win API에서 더블 버퍼링 사용하기
```cpp
void DrawImage::CreateBitmap()
{
	hImg = (HBITMAP)LoadImage(NULL, str,
		IMAGE_BITMAP, 0, 0, LR_LOADFROMFILE | LR_CREATEDIBSECTION);
	GetObject(hImg, sizeof(BITMAP), &bitImg);
}
void DrawImage::DrawBitmapBack(HDC hMemDC, HDC hMemDC2, HBITMAP hOldBitmap2)
{
	int bx, by;
	{
		hMemDC2 = CreateCompatibleDC(hMemDC);
		hOldBitmap2 = (HBITMAP)SelectObject(hMemDC2, hImg);
		bx = bitImg.bmWidth;
		by = bitImg.bmHeight;
		BitBlt(hMemDC, 0, 0, bx, by, hMemDC2, 0, 0, SRCCOPY);
		SelectObject(hMemDC2, hOldBitmap2);
		DeleteDC(hMemDC2);
	}
}
void DrawImage::DrawBitmapFront(HDC hMemDC, HDC hMemDC2, HBITMAP hOldBitmap2,Player line, std::list<DrawObject*> objects, DrawObject randomImage)
{
	int bx, by;
	{
		hMemDC2 = CreateCompatibleDC(hMemDC);
		hOldBitmap2 = (HBITMAP)SelectObject(hMemDC2, hImg);
		bx = bitImg.bmWidth;
		by = bitImg.bmHeight;
		randomImage.CreateRandom(hMemDC2);
		for (std::list<DrawObject*>::iterator it = objects.begin(); it != objects.end(); it++)
		{
			DrawObject *obj = *it;
			if (it != objects.begin())
			{
				obj->CreateObject(line, hMemDC2);
			}
		}
		TransparentBlt(hMemDC, 0, 0, bx, by, hMemDC2, 0, 0,
			bx, by, RGB(255,0,255));
		SelectObject(hMemDC2, hOldBitmap2);
		DeleteDC(hMemDC2);
	}
}

void DrawImage::DeleteBitmap()
{
	DeleteObject(hImg);
}
```

```cpp
 //doubleBuffering
void DrawdoubleBuffering(HDC hdc, HWND hWnd) 
{
 HDC hMemDC = NULL;
    HBITMAP hOldBitmap = NULL;
    //int bx, by;

    HDC hMemDC2 = NULL;
    HBITMAP hOldBitmap2 = NULL;

    //메모리 디씨 생성
    hMemDC = CreateCompatibleDC(hdc); //복사본 만들기
    if (!hMemDC)
    {
        MessageBox(hWnd, _T("CreateCompatibleDC failed!"), _T("Error"), MB_OK);
        return;
    }
    if (hDoubleBufferImage == NULL)
    {
        hDoubleBufferImage = CreateCompatibleBitmap
        (hdc, rectView.right, rectView.bottom);
    }
    
    hOldBitmap = (HBITMAP)SelectObject(hMemDC, hDoubleBufferImage);
    back.DrawBitmapBack(hMemDC, hMemDC2, hOldBitmap2); // bgImg
	
    // 앞 이미지
    front.DrawBitmapFront(hMemDC, hMemDC2, hOldBitmap2,line,objects,object);
    pen.PaintText(hMemDC);//UI
    line.Draw_line(hMemDC);	
    BitBlt(hdc, 0, 0, rectView.right, rectView.bottom, hMemDC, 0, 0, SRCCOPY);
    
    SelectObject(hMemDC, hOldBitmap);
    DeleteDC(hMemDC);
}
```

## 그래픽스 외의 활용법

- 변경 중인 상태에 접근할 수 있다 ← 이중 버퍼로 해결하려는 문제의 핵심
- 원인
    - 다른 스레드가 인터럽트에서 상태에 접근하는 경우
    - 어떤 상태를 변경하는 코드가 동시에 지금 변경하려는 상태를 읽는 경우
        - 인공 지능같이 객체가 서로 상호작용할 때의 경우


