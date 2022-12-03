# wxk-project8

```java
import controlP5.*;
import peasy.*;
PeasyCam cam;
ControlP5 bar;

int wn=20;
int chtime=100;
float next;
float  foods;
int health;
World world;

void setup() {
  size(800, 800);

  world = new World(wn);
  smooth();
  
   cam = new PeasyCam(this, 10);
  UI();
}

void draw() {
 
  
  background(150);
  world.run();

   UIShow();
}


void mousePressed() {
  world.born(mouseX,mouseY); 
}


class Bloop {
  PVector location; 
  DNA dna;          
  float health;     
  float xoff;       
  float yoff;
 
  float r;
  float maxspeed;

 
  Bloop(PVector l, DNA dna_) {
    location = l.get();
    health = 200;
    xoff = random(1000);
    yoff = random(1000);
    dna = dna_;
   
    maxspeed = map(dna.genes[0], 0, 1, 15, 0);
    r = map(dna.genes[0], 0, 1, 0, 50);
  }

  void run() {
    update();
    borders();
    display();
  }

  
  void eat(Food f) {
    ArrayList<PVector> food = f.getFood();
   
    for (int i = food.size()-1; i >= 0; i--) {
      PVector foodLocation = food.get(i);
      float d = PVector.dist(location, foodLocation);
     
      if (d < r/2) {
        health += chtime; 
        food.remove(i);
      }
    }
  }








  Bloop reproduce() {
  
    if (random(1) < next) {

      DNA childDNA = dna.copy();
     
      childDNA.mutate(0.01);
      return new Bloop(location, childDNA);
    } 
    else {
      return null;
    }
  }

 
  void update() {
   float vx = map(noise(xoff),0,1,-maxspeed,maxspeed);
    float vy = map(noise(yoff),0,1,-maxspeed,maxspeed);
    PVector velocity = new PVector(vx,vy);
    xoff += 0.01;
    yoff += 0.01;

    location.add(velocity);
    
    health -= 0.5;
  }

 
  void borders() {
    if (location.x < -r) location.x = width+r;
    if (location.y < -r) location.y = height+r;
    if (location.x > width+r) location.x = -r;
    if (location.y > height+r) location.y = -r;
  }

 
  void display() {
    ellipseMode(CENTER);
    stroke(0,health);
    fill(0, health);
    ellipse(location.x, location.y, r, r);
  }


  boolean dead() {
    if (health < 0.0) {
      return true;
    } 
    else {
      return false;
    }
  }
}







class DNA {

  
  float[] genes;
  
 
  DNA() {
 
    genes = new float[1];
    for (int i = 0; i < genes.length; i++) {
      genes[i] = random(0,1);
    }
  }
  
  DNA(float[] newgenes) {
    genes = newgenes;
  }
  
  DNA copy() {
    float[] newgenes = new float[genes.length];
   
    for (int i = 0; i < newgenes.length; i++) {
      newgenes[i] = genes[i];
    }
    
    return new DNA(newgenes);
  }
  

  void mutate(float m) {
    for (int i = 0; i < genes.length; i++) {
      if (random(1) < m) {
         genes[i] = random(0,1);
      }
    }
  }
}







class Food {
  ArrayList<PVector> food;
 
  Food(int num) {
   
    food = new ArrayList();
    for (int i = 0; i < num; i++) {
       food.add(new PVector(random(width),random(height))); 
    }
  } 
  

  void add(PVector l) {
     food.add(l.get()); 
  }
  

  void run() {
    for (PVector f : food) {
       rectMode(CENTER);
       stroke(0);
       fill(175);
       rect(f.x,f.y,8,8);
    } 
  
    if (random(1) < foods) {
       food.add(new PVector(random(width),random(height))); 
    }
  }
  
 
  ArrayList getFood() {
    return food; 
  }
}







class World {

  ArrayList<Bloop> bloops;   
  Food food;

 
  World(int num) {
  
    food = new Food(num);
    bloops = new ArrayList<Bloop>();             
    for (int i = 0; i < num; i++) {
      PVector l = new PVector(random(width),random(height));
      DNA dna = new DNA();
      bloops.add(new Bloop(l,dna));
    }
  }

 
  void born(float x, float y) {
    PVector l = new PVector(x,y);
    DNA dna = new DNA();
    bloops.add(new Bloop(l,dna));
  }

 
  void run() {

    food.run();
    
  
    for (int i = bloops.size()-1; i >= 0; i--) {
     
      Bloop b = bloops.get(i);
      b.run();
      b.eat(food);
    
      if (b.dead()) {
        bloops.remove(i);
        food.add(b.location);
      }
    
      Bloop child = b.reproduce();
      if (child != null) bloops.add(child);
    }
  }
}



int canvasLeftCornerX = 30;
int canvasLeftCornerY = 60;



void s1() {
  size(500, 500);
 cam = new PeasyCam(this,500);
  UI();
}

void d1() {
  background(51);

  UIShow();
}



void UI() {
  bar = new ControlP5(this, createFont("微软雅黑", 14));
  int barSize = 100;
  int barHeight = 10;
  int barInterval = barHeight + 10;


  bar.addSlider("chtime", 1,1000,100, canvasLeftCornerX, canvasLeftCornerY, barSize, barHeight).setLabel("食物健康数值");
  bar.addSlider("next", 0, 0.003, 0.0001, canvasLeftCornerX, canvasLeftCornerY+barInterval, barSize, barHeight).setLabel("生产概率");
  bar.addSlider("foods", 0, 1, 0.0001, canvasLeftCornerX, canvasLeftCornerY+barInterval*3, barSize, barHeight).setLabel("食物数量");
  bar.addSlider("health", 0, 500, 200, canvasLeftCornerX, canvasLeftCornerY+barInterval*4, barSize, barHeight).setLabel("初始健康值");

  bar.setAutoDraw(false);
}



void UIShow() {
  cam.beginHUD();
 
  bar.draw();
  cam.endHUD();

}
