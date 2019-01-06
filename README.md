# info_ta
Autonomie_1

#collaborateur: JIN Lin & GAO Xinzhao & ZHAO Rui



#Le Principe du jeu
#On dessin des points jaunes en formant une carré 6*6. Chaque point est connecté au moins deux autres points. Initialement, tous les lignes sont croisées ensemble. le joueur déplace le point afin que les lignes ne se croisent pas. Lorsque toutes les lignes ne sont pas entrelacées, le joueur réussit.




from kivy.app import App
from kivy.uix.widget import Widget
from kivy.graphics import Color, Ellipse, Line, InstructionGroup, Rectangle
from kivy.uix.button import Button
from kivy.uix.floatlayout import FloatLayout
from kivy.uix.label import Label
from kivy.uix.scatter import Scatter
from kivy.properties import ListProperty
import random

# g_ballsize = 60
# g_halfballsize = 30
# g_balldistance = 120
# g_initdistance = 100

g_ballsize = 30
g_halfballsize = 15
g_balldistance = 80
g_initdistance = 0


def crosslinecheck(line1, line2):
#Pour déterminer si les lignes se croisent
    ax = line1[0]
    ay = line1[1]
    bx = line1[2]
    by = line1[3]

    cx = line2[0]
    cy = line2[1]
    dx = line2[2]
    dy = line2[3]

    if ax == cx and ay == cy:
        if bx == dx and by == dy:
            return True
        else:
            return False

    if ax == dx and ay == dy:
        if bx == cx and by == cy:
            return True
        else:
            return False

    if bx == cx and by == cy:
        if ax == dx and ay == dy:
            return True
        else:
            return False

    if bx == dx and by == dy:
        if ax == cx and ay == cy:
            return True
        else:
            return False

    if not (min(ax, bx) <= max(cx, dx) and min(cy, dy) <= max(ay, by) and min(cx, dx) <= max(ax, bx) and min(ay,
                                                                                                             by) <= max(
            cy, dy)):
        return False
    u = (cx - ax) * (by - ay) - (bx - ax) * (cy - ay)
    v = (dx - ax) * (by - ay) - (bx - ax) * (dy - ay)
    w = (ax - cx) * (dy - cy) - (dx - cx) * (ay - cy)
    z = (bx - cx) * (dy - cy) - (dx - cx) * (by - cy)
    return (u * v <= 0.00000001 and w * z <= 0.00000001)



class Ball(Label):
#Dessiner une balle
    def __init__(self, **kwargs):
        super(Ball, self).__init__(**kwargs)
        with self.canvas:
            Color(1, 1, 0)
            Ellipse(size=(g_ballsize, g_ballsize))

    def on_touch_down(self, touch):
        self.canvas.clear()
        with self.canvas:
            Color(0, 1, 0)
            Ellipse(size=(g_ballsize, g_ballsize))

    def on_touch_up(self, touch):
        self.canvas.clear()
        with self.canvas:
            Color(1, 1, 0)
            Ellipse(size=(g_ballsize, g_ballsize))


class MyScatter(Scatter):
#Dessiner lignes entre balles
    lineproperties = ListProperty([])

    def __init__(self, index=0, **kwargs):
        self.index = index
        super(MyScatter, self).__init__(**kwargs)
        self.do_rotation = False
        self.do_scale = False
        self.add_widget(Ball())
        self.lineproperties = [index, self.pos]

    def setIndex(self, index):
        self.index = index

    def on_pos(self, instance, pos):
        if len(self.lineproperties) > 1:
            self.lineproperties[1] = (self.pos[0] + g_halfballsize, self.pos[1] + g_halfballsize)


class MainWidget(Widget):
    linedict = {}

    def __init__(self, **kwargs):

        super(MainWidget, self).__init__(**kwargs)
        ballnumber = 36
        linepair = self.reduceline(self.initBallsLines(ballnumber), ballnumber)
        ballposdict = self.randomBallPos(ballnumber)


        for k in ballposdict:
            scat = MyScatter(index=k, size=(g_ballsize, g_ballsize), pos=ballposdict[k])
            scat.bind(lineproperties=self.writelines)
            self.add_widget(scat)

        self.combineLineAndPos(linepair, ballposdict)

        for k in self.linedict:
            pairlist = self.linedict[k]
            self.obj = InstructionGroup()
            self.obj.add(Color(1, 0, 1))
            self.obj.add(Line(points=(pairlist[0][1], pairlist[1][1])))
            self.linedict[k].append(self.obj)
            self.canvas.add(self.obj)

    def initBallPos(self, number):
        colnum = int(number ** 0.5)

        ballposdict = {}

        for count in range(number):
            posx = int(count / colnum)
            posy = count % colnum

            ballposdict[count] = (posx * g_balldistance + g_initdistance, posy * g_balldistance + g_initdistance)
        return ballposdict

    def randomBallPos(self, number):
        indexlist = list(range(number))

        random.shuffle(indexlist)

        ballposdict = self.initBallPos(number)

        newballposdict = {}
        for k in range(number):
            newballposdict[indexlist[k]] = ballposdict[k]

        return newballposdict

    def combineLineAndPos(self, linepair, ballposdict):

        for lineindex in linepair:
            firstball = linepair[lineindex][0]
            secondball = linepair[lineindex][1]
            firstpos = (ballposdict[firstball][0] + g_halfballsize, ballposdict[firstball][1] + g_halfballsize)
            secondpos = (ballposdict[secondball][0] + g_halfballsize, ballposdict[secondball][1] + g_halfballsize)
            self.linedict[lineindex] = [[firstball, firstpos], [secondball, secondpos]]

    def initBallsLines(self, number):
        colnum = int(number ** 0.5)
        linepair = {}
        linecount = 0

        for point in range(number):
            if point + 1 < number and (point + 1) % colnum != 0:
                linepair[linecount] = (point, point + 1)
                linecount += 1
            # Imprimer la ligne horizontale
            if point + colnum < number:
                linepair[linecount] = (point, point + colnum)
                linecount += 1
            if point + colnum + 1 < number and (point + colnum + 1) % colnum != 0:
                linepair[linecount] = (point, point + colnum + 1)
                linecount += 1
        if (number - 1) % colnum == 0:
            linepair[linecount] = (number - 1, number - 2)

        return linepair

    def reduceline(self, linepair, number):
        pointlines = {}
        for i in range(number):
            pointlines[i] = 0

        for k in linepair:
            p1 = linepair[k][0]
            p2 = linepair[k][1]

            pointlines[p1] += 1
            pointlines[p2] += 1

        newlinepair = {}
        newcount = 0

        for k in linepair:
            p1 = linepair[k][0]
            p2 = linepair[k][1]

            if pointlines[p1] > 4 and pointlines[p2] > 4:
                pointlines[p1] -= 1
                pointlines[p2] -= 1
            else:
                newlinepair[newcount] = (p1, p2)
                newcount += 1

        return newlinepair

    def rewriteColor(self):
        for k in self.linedict:
            if self.checkLineCross(k):
                #imprimer ligne en rouge
                self.canvas.remove(self.linedict[k][2])
                self.obj = InstructionGroup()
                self.obj.add(Color(1, 0, 0))
                self.obj.add(Line(points=(self.linedict[k][0][1], self.linedict[k][1][1])))
                self.linedict[k][2] = self.obj
                self.canvas.add(self.linedict[k][2])
            else:
                #imprimer ligne en vert
                self.canvas.remove(self.linedict[k][2])
                self.obj = InstructionGroup()
                self.obj.add(Color(0, 1, 0))
                self.obj.add(Line(points=(self.linedict[k][0][1], self.linedict[k][1][1])))
                self.linedict[k][2] = self.obj
                self.canvas.add(self.linedict[k][2])

    def checkLineCross(self, lineindex):
        ball1 = self.linedict[lineindex][0][1]
        ball2 = self.linedict[lineindex][1][1]
        srcline = [ball1[0], ball1[1], ball2[0], ball2[1]]
        for k in self.linedict:
            if k == lineindex:
                continue
            ball1 = self.linedict[k][0][1]
            ball2 = self.linedict[k][1][1]
            destline = [ball1[0], ball1[1], ball2[0], ball2[1]]

            if crosslinecheck(srcline, destline):
                return True

        return False

    def writelines(self, instance, value):

        ballindex = value[0]
        ballpos = value[1]

        changeflag = False
        for k in self.linedict:
            if self.linedict[k][0][0] == ballindex:
                self.linedict[k][0][1] = ballpos
                changeflag = True
            elif self.linedict[k][1][0] == ballindex:
                self.linedict[k][1][1] = ballpos
                changeflag = True

            if changeflag:
                self.canvas.remove(self.linedict[k][2])
                self.obj = InstructionGroup()
                self.obj.add(Line(points=(self.linedict[k][0][1], self.linedict[k][1][1])))
                self.linedict[k][2] = self.obj
                self.canvas.add(self.obj)
        self.rewriteColor()

class MyApp(App):
    def build(self):
        return MainWidget()


if __name__ == '__main__':
    MyApp().run()
