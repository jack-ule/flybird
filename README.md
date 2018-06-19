# flybird
小鸟飞翔
主体部分
package com.bdqn.Bird;

import java.awt.Color;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyListener;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;
import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;
import javax.swing.JFrame;
import javax.swing.JPanel;
/*游戏面板*/
public class BirdGame extends JPanel{
	/*小鸟*/
	private Bird bird;
	/*地面*/
	private Ground ground;
	/*第一根柱子*/
	private Column column1;
	/*第二根柱子*/
	private Column column2;
	/*背景图片*/
	private BufferedImage background;
	//定义分数
	private int score;
	boolean gameOver;
	private BufferedImage gameOverImage;
	private int state;
	private static final int START=0;
	private static final int RUNNING=1;
	private static final int GAME_OVER=2;
	private BufferedImage startImage;

	/*初始化游戏内容*/
	public BirdGame() throws IOException {
		gameOver=false;
		gameOverImage=ImageIO.read(getClass().getResource("gameover.png"));
		/*初始小鸟*/
		bird=new Bird();
		/*初始第一根柱子*/
		column1=new Column(1);
		/*初始第二根柱子*/
		column2=new Column(2);
		/*初始地面*/
		ground=new Ground();
		state=START;
		startImage= ImageIO.read(getClass().getResource("start.png"));

		/*初始背景图片*/
		background=ImageIO.read(getClass().getResource("bg.png"));
	}
	
	/*重写paint方法,绘制游戏界面*/
	public void paint(Graphics g) {
		super.paint(g);
		/*绘制背景*/
		g.drawImage(background, 0, 0, null);
		/*绘制第一根柱子*/
		g.drawImage(column1.getColumnImage(), column1.getX()-column1.getWidth()/2, 
				column1.getY()-column1.getHeight()/2, null);
		/*绘制第二根柱子*/
		g.drawImage(column2.getColumnImage(), column2.getX()-column2.getWidth()/2, 
				column2.getY()-column2.getHeight()/2, null);
		/*绘制地面*/
		g.drawImage(ground.getGroundImage(), ground.getX(), ground.getY(), null);
		/*绘制小鸟*/
		//追加旋转动作
		Graphics2D g2=(Graphics2D) g;
		g2.rotate(-bird.getAlpha(),bird.getX(),bird.getY());
		g.drawImage(bird.getBirdImage(), bird.getX()-bird.getWidth()/2, 
				bird.getY()-bird.getHeight()/2, null);
		g2.rotate(bird.getAlpha(),bird.getX(),bird.getY());
		//绘制分数
		Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
		g.setFont(f);
		g.drawString(""+score, 40, 60);
		//绘制重影
		g.setColor(Color.BLUE);
		g.drawString(""+score, 40-3, 60-3);
		switch (state) {
		case GAME_OVER:
			g.drawImage(gameOverImage, 0, 0, null);
			break;
		case START:
			g.drawImage(startImage, 0, 0, null);
			break;
		}


	}
	//画面运动,控制一系列界面移动
	public void action() {
		new Thread() {
			public void run() {
				while(true) {
					if(!gameOver) {
						switch (state) {
						case START:
							bird.fly();
							ground.step();
							break;
						case RUNNING:
							column1.step();
							column2.step();
							bird.step();//上下移动
							bird.fly();//挥动翅膀
							ground.step();//地面移动
						}

					}
					/*碰撞地面*/
					if(bird.hit(ground)) {
						gameOver=true;
					}

					//地面移动
//					ground.step();
//					//柱子移动
//					column1.step();
//					column2.step();
//					//实现鸟的上抛
//					bird.step();
//					//鸟的挥动翅膀
//					bird.fly();
				//增加鼠标监听事件
					if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
						state=GAME_OVER;
					}


					MouseListener listener=new MouseAdapter() {

						@Override
						public void mousePressed(MouseEvent e) {
							// TODO Auto-generated method stub
							try {
								switch (state) {
								case GAME_OVER:
									bird=new Bird();
									column1=new Column(1);
									column2=new Column(2);
									score=0;
									state=START;
									break;
								case START:
									state=RUNNING;
									break;
								case RUNNING:
									//鸟向上飞扬
									bird.flappy();
								}
							} catch (Exception e2) {
								e2.printStackTrace();
							}

						}
						
					};
					//将鼠标监听挂载到面板 监听和面板连接到一起
					addMouseListener(listener);
//					KeyListener listener2=new KeyAdapter() {
//						键盘监听
//					};
					//增加记分逻辑
					if(bird.getX()==column1.getX()||bird.getX()==column2.getX()) {
						score++;
					}
					//刷新界面
					repaint();
					//间隔多少秒移动一次
					try {
						Thread.sleep(1000/30);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}.start();
		
	}
	/*游戏启动*/
	public static void main(String[] args) throws IOException {
		/*构建窗口*/
		JFrame frame=new JFrame();
		/*构建游戏面板*/
		BirdGame birdGame=new BirdGame();
		//调用界面运动
		birdGame.action();
		/*将面板添加到窗口*/
		frame.add(birdGame);
		frame.setSize(864/2, 644);
		frame.setLocationRelativeTo(null);
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		frame.setVisible(true);
	}
}
鸟部分
package com.bdqn.Bird;
/*
 * 小鸟
 */

import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Bird {
	/*小鸟中心点x坐标*/
	private int x;
	/*小鸟中心点y坐标*/
	private int y;
	/*小鸟的宽度*/
	private int width;
	/*小鸟的高度*/
	private int height;
	/*小鸟的尺寸---用于后续碰撞检测*/
	private int size;
	/*小鸟的背景图片*/
	private BufferedImage birdImage;
	private double g;
	private double v0;
	private double speed;
	private double s;
	private double t;
	private double alpha;//旋转角度
	//定义鸟的动画帧
	private BufferedImage[] images;
	private int index;
	
	/*初始化小鸟*/
	public Bird() throws IOException {
		birdImage=ImageIO.read(getClass().getResource("0.png"));
		x=132;
		y=280;
		width=birdImage.getWidth();
		height=birdImage.getHeight();
		size=40;
		//初始上抛属性
		g=4;
		v0=20;
		t=0.25;
		s=0;
		speed=v0;
		images=new BufferedImage[8];
		for(int i=0;i<8;i++) {
			images[i]=ImageIO.read(getClass().getResource(i+".png"));
			
		}
		index=0;
		alpha=0;
	}
	//实现小鸟上抛
	public void step() {
		double v0=speed;
		s=v0*t+g*t*t/2;
		y=y-(int)s;
		double v=v0-g*t;
		speed=v;
		//更新倾角
		alpha=Math.atan(s/8);
	}
	//重置上抛速度
	public void flappy() {
		speed=v0;//当前速度重置为初始速度
	}
	public double getAlpha() {
		return alpha;
	}
	public void setAlpha(double alpha) {
		this.alpha = alpha;
	}
	public void fly() {
		//切换变慢（index/12）%8
		index++;
		
		birdImage=images[(index/12)%8];
	}
	public boolean hit(Column column) {
		/*先检测是否在柱子的范围以内*/
		if(x>column.getX()-column.getWidth()/2-size/2 && 
				x<column.getX()+column.getWidth()/2+size/2) {
				/*检测是否在柱子缝隙中*/
			if(y>column.getY()-column.getGap()/2+size/2 && 
					y<column.getY()+column.getGap()/2-size/2) {
				return false;
			}
			return true;
		}
		return false;
	}
	public boolean hit(Ground ground) {
		boolean hit = y+size/2>ground.getY();
		if(hit) {
			/*将鸟放置到地面上*/
			y=ground.getY()-size/2;
			/*使鸟摔倒在地面上有摔倒效果*/
			alpha= -3.14159265358979323/2;
		}
		return hit;
	}

	
	public int getX() {
		return x;
	}
	public int getY() {
		return y;
	}
	public int getWidth() {
		return width;
	}
	public int getHeight() {
		return height;
	}
	public int getSize() {
		return size;
	}
	public BufferedImage getBirdImage() {
		return birdImage;
	}
	
	
}
柱子部分
package com.bdqn.Bird;
/*
 * 柱子
 */

import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Random;

import javax.imageio.ImageIO;

public class Column {
	/*柱子中心点x坐标*/
	private int x;
	/*柱子中心点y坐标*/
	private int y;
	/*柱子的宽度*/
	private int width;
	/*柱子的高度*/
	private int height;
	/*上下柱子间的间距*/
	private int gap;
	/*相邻柱子的间距*/
	private int distance;
	/*背景图片*/
	private BufferedImage columnImage;
	/*随机数字类*/
	private Random random=new Random();
	
	/*初始化柱子,n表示第几根柱子*/
	public Column(int n) throws IOException {
		columnImage=ImageIO.read(getClass().getResource("column.png"));
		width=columnImage.getWidth();
		height=columnImage.getHeight();
		gap=144;
		distance=245;
		x=550+(n-1)*distance;
		System.out.println(x);
		y=random.nextInt(218)+132;
		System.out.println(y);
	}
	//柱子移动
	public void step() {
		//柱子向左移动
		x--;
		if(x==-width/2) {
			//柱子重新回到出场位置
			x=2*distance-width/2;
			y=random.nextInt(218)+132;
		}
	}
	
	public int getX() {
		return x;
	}
	public int getY() {
		return y;
	}
	public int getWidth() {
		return width;
	}
	public int getHeight() {
		return height;
	}
	public int getGap() {
		return gap;
	}
	public int getDistance() {
		return distance;
	}
	public BufferedImage getColumnImage() {
		return columnImage;
	}
}
地面部分
package com.bdqn.Bird;
/*
 * 地面
 */

import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Ground {
	/*地面的x坐标*/
	private int x;
	/*地面的y坐标*/
	private int y;
	/*地面的宽度*/
	private int width;
	/*地面的高度*/
	private int height;
	/*背景图片*/
	private BufferedImage groundImage;
	
	/*初始化地面*/
	public Ground() throws IOException {
		groundImage=ImageIO.read(getClass().getResource("ground.png"));
		x=0;
		y=500;
		width=groundImage.getWidth();
		height=groundImage.getHeight();
	}
	//地面移动
	public void step() {
		//地面向左移动
		x--;
		if(x==-109) {
			x=0;
		}
	}
	
	public int getX() {
		return x;
	}
	public int getY() {
		return y;
	}
	public int getWidth() {
		return width;
	}
	public int getHeight() {
		return height;
	}
	public BufferedImage getGroundImage() {
		return groundImage;
	}
}








