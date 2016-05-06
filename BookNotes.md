
###1. Odoo开发--Getting started with Odoo development  链接

         http://www.jeffzhang.cn/Odoo-Notes-1/
         
 
###2.启动postgresSQL
       pg_ctl -D/usr/local/var/postgres start
       
       
     
###3.Odoo的model里的修饰器
    （1）@api.depends()  作用在定义需要计算的字段里定义的函数，用来指定哪些字段参与计算    
     taken_seats = fields.Float(string="Taken seats", compute='_taken_seats')
 
    @api.depends('seats', 'attendee_ids')
    def _taken_seats(self):
        for r in self:
            if not r.seats:
                r.taken_seats = 0.0
            else:
                r.taken_seats = 100.0 *   
                len(r.attendee_ids) / r.seats
         
     （2）@onchange()作用在指定的字段，当指定字段的值发生变化的时
          候触发被修饰的函数执行
          @api.onchange('seats', 'attendee_ids')
          def _verify_valid_seats(self):
             if self.seats < 0:
                  return {
                    'warning': {
                    'title': "Incorrect 'seats' value",
                    'message': "The number of available    
                    seats may not be negative",
                },
            }
             if self.seats < len(self.attendee_ids):
                 return {
                'warning': {
                    'title': "Too many attendees",
                    'message': "Increase seats or remove 
                    excess attendees",
                },
            }
            
     （3）@api.constrains()约束修饰器，用来校验
     @api.constrains('instructor_id', 'attendee_ids')
      def _check_instructor_not_in_attendees(self):
        for r in self:
            if r.instructor_id and r.instructor_id in 
            	r.attendee_ids:
             raise exceptions.ValidationError("A 
             session's instructor can't be an attendee")
 
     （4）@api.multi重写父类函数
     
###4.数据库表与表的关系
	（1）many2one是用来建立两个表之间的关联的，必须在子表里定义一个字段（实体表里也会生成这个字段），指向主表的model。
    例如：course和session的关联里，session子表的model里就要定义一个course_id，指向主表的一条记录。
    一个session里只能有一个course，一个course里有过的session。
    course_id = fields.Many2one('openacademy.course',
    ondelete='cascade', string="Course", required=True)
 
    （2）one2many是一个虚拟关系，定义了也不会在实体表里创建字段的。在主表里定义，指向明细表的model，并且必须指定明细表里定义的和主表相关联的字段。必须先定义many2one之后才能定义one2many。
    例如：course主表里可以定义一个session_ids代表子表的一个集合。
    session_ids = fields.One2many(
        'openacademy.session', 'course_id', string="Sessions")
 
    （3）many2many会创建一个两个实体表的主键的新的关联表。关联表为两个实体表名加_rel。
    例如：session里定义一个出席者的字段。一个session里可以有多个  Attendees，一个attendee也存在于多个session里，这时就要定义两个表的关联关系表。
    attendee_ids = fields.Many2many('res.partner', string="Attendees")2
        
     