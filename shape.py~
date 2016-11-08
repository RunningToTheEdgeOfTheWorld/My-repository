#!/bin/env python
# -*- coding: UTF-8 -*-
import wx
import time
import random
import copy
import threading
import cPickle as pickle
import os
COLOR_DIC={0:"#CDC6BC",2:"#FEEFDC",4:"#FEE7BB",8:"#FF8F6A",16:"#FE7251",32:"#FF6241",64:"#FE4600",128:"#EDE082",256:"#EDCA00",512:"#EDAE07",1024:"#ED9A00",2048:"#FFD601"} #设置值的背景色
SQUARE_LEVEL=4
length=400/SQUARE_LEVEL
edge_distance=length/10.0
SCREEN_LONG=(length+edge_distance)*(SQUARE_LEVEL+2)+edge_distance #画面宽度，推荐与长度相同
SCREEN_HIGH=(length+edge_distance)*SQUARE_LEVEL+edge_distance #画面长度
RANDOM_NUM=1
INIT_RANDOM_NUM=SQUARE_LEVEL-1

class Point(object):
    def __init__(self,x,y):
        self.x=int(x)
        self.y=int(y)
        self.xy=(x,y)
    def __add__(self,other):
        return (self.x+other.x,self.y+other.y)

    def __sub__(self,other):
        return (self.x-other.x,self.y-other.y)


class Shape(object):
    def __init__(self,pos,points,color):
        point_tuple=[]
        for point in points:
            assert isinstance(point,Point),"输入的不是点对象"
            point_tuple.append(point.xy)
        self.pos=pos
        self.point_tuple=tuple(point_tuple)
        self.color=color
    def area(self):
        raise ImportError



class Square(Shape):
    def __init__(self,pos,value,length,color=""):
        self.length=length
        points=[Point(pos[0],pos[1]),Point(pos[0]+length,pos[1]),Point(pos[0]+length,pos[1]+length),Point(pos[0],pos[1]+length),Point(pos[0],pos[1])]
        self.value=value
        if color=="":
            color=COLOR_DIC[value]
        Shape.__init__(self,pos,points,color)
    def area(self):
        return self.length**2

class MainFrame(wx.Frame):
    def __init__(self):
        wx.Frame.__init__(self, None, -1,"My Frame", size=(SCREEN_LONG+18,SCREEN_HIGH+97))
        self.Bind(wx.EVT_PAINT,self.OnPaint)
        self.Bind(wx.EVT_KEY_DOWN,self.Calculate)
        self.SetBackgroundColour("#CEB69A")
        self.status_list=[[0]*SQUARE_LEVEL for i in xrange(SQUARE_LEVEL)]
        self.score=0
        self.sound1 = wx.Sound(u'success.wav')
        self.sound2 = wx.Sound(u'failed.wav')
        self.CreateStatusBar()
        self.menubar=wx.MenuBar()
        menu_game=wx.Menu()
        menu_1=menu_game.Append(wx.NewId(),u"&新游戏<enter>",u"开始一个新游戏")
        menu_2=menu_game.Append(wx.NewId(),u"设置<s>",u"对游戏内容进行配置")
        menu_3=menu_game.Append(wx.NewId(),u"排行榜<p>",u"查看游戏的历史战绩")
        menu_4=menu_game.Append(wx.NewId(),u"退出<alt+f4>",u"退出游戏")
        
        self.menubar.Append(menu_game,u"&游戏")
        menu_help=wx.Menu()
        menu_5=menu_help.Append(wx.NewId(),u"游戏规则",u"点击查看游戏规则")
        self.menubar.Append(menu_help,u"&帮助")
        self.SetMenuBar(self.menubar)
        self.ranking_obj=RankingList()
        self.rank_message=wx.MessageDialog(self,u"第一名：{0[0]}:{0[1]}\n第二名：{1[0]}:{1[1]}\n第三名：{2[0]}:{2[1]}".format(*self.ranking_obj.ranking_list),u"排行榜",wx.OK)
        self.set_dialog=SettingDialog()
        self.Bind(wx.EVT_MENU,self.NewGame,menu_1)
        self.Bind(wx.EVT_MENU,self.Settings,menu_2)
        self.Bind(wx.EVT_MENU,self.ShowRankMessage,menu_3)
        self.Bind(wx.EVT_MENU,self.ExitGame,menu_4)
        self.Bind(wx.EVT_MENU,self.ShowHelp,menu_5)
        self.Centre()

    def ShowHelp(self,event):
        wx.MessageBox(u"游戏规则:\n    滑动屏幕来移动小方块，两个数字一样的小方块相撞时就会相加合成一个方块，每次操作之后会在空白的方格处随机生成一个2或4的方块，最终得到一个2048的方块就算胜利了，如果16个格子全部填满无法移动的话GAMEOVER。")

    def Settings(self,enent):
        self.set_dialog.Centre()
        if self.set_dialog.ShowModal()==wx.ID_OK:
            pass


    def ShowRankMessage(self,event):
        self.rank_message.ShowModal()

    def RandomValue(self,number=RANDOM_NUM):
        """状态List生成随机值"""
        random_value_dic={}
        try:
            rand=random.sample(self.GetEmptyList(),number)
            for rand_i in rand:
                self.status_list[rand_i[0]][rand_i[1]]=random.choice((2,2,2,2,2,2,2,2,4))
        except ValueError:
            wx.MessageBox(u"游戏结束！你的分数是:%s"%self.score)
            self.Close()
            time.sleep(1)
        
    def OnPaint(self,event=None):
        print 'Onpaint!'
        square_head=Square(pos=(SCREEN_HIGH,edge_distance),value=2048,length=length*2+edge_distance,color="#FFD601")#创建要显示的正方形
        self.rev=wx.StaticText(parent=self,id=-1,label=u"\n你的分数\n%s\n"%self.score,pos=(SCREEN_HIGH,edge_distance*2+square_head.length),size=(length,length),style=wx.ALIGN_CENTER)
        self.rev.SetBackgroundColour(COLOR_DIC[0])
        self.rev.SetForegroundColour('white')
        font=wx.Font(13,wx.DEFAULT,wx.NORMAL,wx.BOLD,faceName=u"黑体")
        self.rev.SetFont(font)
        self.DrawSquare(square_head)
        for index_line,one_row in enumerate(self.status_list):
            for index_column,status in enumerate(one_row):
                pos=(edge_distance+index_column*(length+edge_distance),edge_distance+index_line*(length+edge_distance))
                square1=Square(pos=pos,value=status,length=length)#创建要显示的正方形
#               time.sleep(0.1)
                self.DrawSquare(square1)
#               thread_draw=threading.Thread(target=self.DrawSquare,args=(square1,))
#               thread_draw.start()

    def ExitGame(self,event):
        self.Close()

    def NewGame(self,event=None):
        self.status_list=[[0]*SQUARE_LEVEL for i in xrange(SQUARE_LEVEL)]
        self.score=0
        self.RandomValue(INIT_RANDOM_NUM)
        if event:
            self.OnPaint()

    def Calculate(self,event):
        """根据输入方向键转置矩阵"""
        key=event.GetKeyCode()
        old_status=copy.deepcopy(self.status_list)
        if key==13:#回车
            self.NewGame()
        elif key==314:#左
            self.LeftMove()
        elif key==316:#右
            self.status_list=[row[::-1] for row in self.status_list]
            self.LeftMove()
            self.status_list=[row[::-1] for row in self.status_list]
        elif key==315:#上
            self.status_list=map(list,zip(*self.status_list))
            self.LeftMove()
            self.status_list=map(list,zip(*self.status_list))
        elif key==317:#下
            self.status_list=map(list,zip(*self.status_list))
            self.status_list=[row[::-1] for row in self.status_list]
            self.LeftMove()
            self.status_list=[row[::-1] for row in self.status_list]
            self.status_list=map(list,zip(*self.status_list))
        else:
            pass
        self.PaintChange(self.GetChangeDict(old_status,self.status_list))
        self.rev.SetLabel(u"\n你的分数\n%s\n"%self.score)
        self.Judgement()


    def LeftMove(self):
        """左移状态变换"""
        old_status=copy.deepcopy(self.status_list)
        for index,raw in enumerate(self.status_list):
            raw=[element for element in raw if element!=0]
            for i in xrange(len(raw)-1):
                if raw[i]==raw[i+1]:
                    raw[i]=raw[i]*2
                    self.score+=raw[i]
                    raw[i+1]=0
            raw=[element for element in raw if element!=0]
            self.status_list[index]=raw+[0]*(SQUARE_LEVEL-len(raw))
        if old_status!=self.status_list:
            self.RandomValue()
            self.sound1.Play(wx.SOUND_ASYNC)
        else:
            self.sound2.Play(wx.SOUND_ASYNC)

    def GetEmptyList(self):
        """得到剩余空格的坐标List"""
        empty_list=[]
        for index1,row in enumerate(self.status_list):
            for index2,element in enumerate(row):
                if element==0:
                    empty_list.append([index1,index2])
        return empty_list

    def GetChangeDict(self,old_status,new_status):
        change_dic={}
        for index1,row in enumerate(old_status):
            for index2,state in enumerate(row):
                if state!=new_status[index1][index2]:
                    change_dic[(index1,index2)]=new_status[index1][index2]
        return change_dic

    def PaintChange(self,change_dic):
        for vector in change_dic:
            pos=(edge_distance+vector[1]*(length+edge_distance),edge_distance+vector[0]*(length+edge_distance))
            square1=Square(pos=pos,value=change_dic[vector],length=length)#创建要显示的正方形
#           self.DrawSquare(square1)
            thread_draw=threading.Thread(target=self.DrawSquare,args=(square1,))
            thread_draw.start()

    def DrawSquare(self,square1,show_value=True):
        lock=threading.Lock()
        brush=wx.Brush(square1.color)
        dc=wx.ClientDC(self)
        dc.SetPen(wx.TRANSPARENT_PEN)
        dc.SetBrush(brush)
        a=(square1.value in [1024,2048]) and 0.7 or 1
        dc.SetFont(wx.Font(square1.length*9.0/30*a,wx.DEFAULT,wx.NORMAL,wx.BOLD,faceName="Arial Black"))
        if square1.value!=0:
#           dc.DrawRectangle(square1.pos[0],square1.pos[1],square1.length,square1.length)
            for i in xrange(int(square1.length)):
                t1=time.time()
                time.sleep(0.001)
                t2=time.time()
                dc.DrawRectangle(square1.pos[0]+(square1.length-i)/2,square1.pos[1]+(square1.length-i)/2,i,i)
            if show_value:
                num_long,num_high=dc.GetTextExtent(str(square1.value))
                lock.acquire()
                f_color='#616161' if square1.value in [2,4] else 'white'
                self.SetForegroundColour(f_color) 
                dc.DrawText(str(square1.value),(square1.length-num_long)/2.0+square1.pos[0],(square1.length-num_high)/2.0+square1.pos[1])
                lock.release()
        else:
            dc.DrawRectangle(square1.pos[0],square1.pos[1],square1.length,square1.length)

    def Judgement(self):
        if len(self.GetEmptyList())==0:
            for index1 in xrange(SQUARE_LEVEL-1):
                for index2 in xrange(SQUARE_LEVEL-1):
                    if self.status_list[index1][index2]==self.status_list[index1+1][index2] or self.status_list[index1][index2]==self.status_list[index1][index2+1]:
                        return False
#           wx.MessageBox(u"游戏结束！你的分数是:%s"%self.score)
            end_dlg=wx.TextEntryDialog(None,u"游戏结束！你的分数是:%s"%self.score,u"游戏结束！",u"你的名字")
            if end_dlg.ShowModal()==wx.ID_OK:
                name=end_dlg.GetValue()
                self.ranking_obj.Insert(name,self.score)
#               f=open("ranking_dict.txt","r")
#               ranking_dict=pickle.load(f)
#               f.close()
#               if self.score>min(ranking_dict.values()):
#                   ranking_dict[name]=self.score
#                   if len(ranking_dict)>3:
#                       for i in ranking_dict:
#                           if ranking_dict[i]==min(ranking_dict.values()):
#                               ranking_dict.pop(i)
#                               break
#               f=open("ranking_dict.txt","w")
#               pickle.dump(ranking_dict,f)
#               f.close()
            self.Close()

class RankingList(object):
    def __init__(self):
        if os.path.exists("ranking_list.txt"):
            f=open('ranking_list.txt','r')
            self.ranking_list=pickle.load(f)
            f.close()
        else:
            f=open("ranking_list.txt",'w')
            self.ranking_list=[[u"<用户名>",0],[u"<用户名>",0],[u"<用户名>",0]]
            pickle.dump(self.ranking_list,f)
            f.close()

    def Insert(self,name,score):
        for index,name_score in enumerate(self.ranking_list):
            if score>name_score[1]:
                self.ranking_list[index]=[unicode(name),score]
                f=open("ranking_list.txt",'w')
                pickle.dump(self.ranking_list,f)
                f.close()
                break

class NotEmptyValidator(wx.PyValidator):
    def __init__(self):
        wx.PyValidator.__init__(self)

    def Clone(self):
        return NotEmptyValidator()

    def Validate(self, win):
        textCtrl = self.GetWindow()
        text = textCtrl.GetValue()

        if len(text) == 0:
            wx.MessageBox("this ﬁeld must contain some text!", "error")
            textCtrl.SetBackgroundColour("pink")
            textCtrl.SetFocus()
            textCtrl.Refresh()
            return False
        else:
            textCtrl.SetBackgroundColour(
            wx.SystemSettings_GetColour(wx.SYS_COLOUR_WINDOW))              
            textCtrl.Refresh()
            return True

    def TransferToWindow(self):
        return True 

    def TransferFromWindow(self):
        return True


class SettingDialog(wx.Dialog):
    def __init__(self):
        wx.Dialog.__init__(self,None,-1,u"设置")
        helpping=u"调整游戏阶数和随机出现的数字块的个数，体验新的玩法！"

        # Create the text controls
        about = wx.StaticText(self, -1,helpping)
        name_l  = wx.StaticText(self, -1,u"游戏阶数:")      
        email_l = wx.StaticText(self, -1,u"每次生成的数量:")
        phone_l = wx.StaticText(self, -1,u"是否显示动画:")

        name_t  = wx.TextCtrl(self, validator=NotEmptyValidator())         
        email_t = wx.TextCtrl(self, validator=NotEmptyValidator())         
        phone_t = wx.TextCtrl(self, validator=NotEmptyValidator())
        # Use standard button IDs
        okay   = wx.Button(self, wx.ID_OK)
        okay.SetDefault()
        cancel = wx.Button(self, wx.ID_CANCEL)

        # Layout with sizers
        sizer = wx.BoxSizer(wx.VERTICAL)
        sizer.Add(about, 0, wx.ALL, 5)
        sizer.Add(wx.StaticLine(self), 0, wx.EXPAND|wx.ALL, 5)         
        fgs = wx.FlexGridSizer(3, 2, 5, 5)
        fgs.Add(name_l, 0, wx.ALIGN_RIGHT)
        fgs.Add(name_t, 0, wx.EXPAND)
        fgs.Add(email_l, 0, wx.ALIGN_RIGHT)
        fgs.Add(email_t, 0, wx.EXPAND)
        fgs.Add(phone_l, 0, wx.ALIGN_RIGHT)
        fgs.Add(phone_t, 0, wx.EXPAND)
        fgs.AddGrowableCol(1)
        sizer.Add(fgs, 0, wx.EXPAND|wx.ALL, 5)

        btns = wx.StdDialogButtonSizer()
        btns.AddButton(okay)
        btns.AddButton(cancel)
        btns.Realize()
        sizer.Add(btns, 0, wx.EXPAND|wx.ALL, 5)

        self.SetSizer(sizer)
        sizer.Fit(self)


if __name__ == '__main__':
    app = wx.PySimpleApp()
    frame = MainFrame()
    frame.Show(True)
    app.MainLoop()
