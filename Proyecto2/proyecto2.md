# Modelo métrica
    # -*- coding: utf-8 -*-

    from odoo import models, fields, api
    from datetime import timedelta


    class subscription(models.Model):
        _name = 'subscription.metricas'
        _description = 'subscription.metricas'

        fecha = fields.Date(string='fecha')
        active_subscriptions = fields.Integer(string='subscipciones activas')
        income_generated= fields.Float(string='ingresos generados')
        renewal_rate = fields.Float(string='tasa de renovación', compute= "_tasa_renovacion" ) # (RENOVACIONES / SUBS ACTIVAS)*100
        cancellation_fee = fields.Float(string='Tasa de cancelación', compute="_tasa_cancelacion") # (CANCELACIONES / SUBS ACTIVAS)*100
        new_subscriptions = fields.Integer(string='nuevas subscripciones')
        canceled_subscriptions= fields.Integer(string='subscripciones canceladas')
        returning_customers= fields.Integer(string='clientes recurrentes')
        new_customers= fields.Integer(string='nuevos clientes')
        arpu = fields.Float(string='ingreso por usuario', compute="_ver_arpu") # INGRESOS GENERADOS (arpu) / SUBS ACTIVAS
        conversion_rate= fields.Float(string='tasa de conversión')
        churn_rate= fields.Float(string='tasa de perdida de clientes') 
        lifetime_value = fields.Float(string='valor generacion cliente') 
        cac = fields.Float(string='costo adquisición clientes')
        note = fields.Text(string='notas')
        subscription_ids= fields.One2many('subscription.subscription', 'metricas_id', string='Subscripciones relacionadas')
        
        @api.depends('renewal_rate', 'active_subscriptions')
        def _tasa_renovacion(self):
            for a in self:
                if (a.active_subscriptions != 0 and a.canceled_subscriptions <= a.active_subscriptions):
                    a.renewal_rate = (1-(a.canceled_subscriptions / a.active_subscriptions)) * 100
                else:
                    a.renewal_rate = 0.0
        

        @api.depends('cancellation_fee', 'active_subscriptions')
        def _tasa_cancelacion(self):
            for a in self:
                if (a.active_subscriptions != 0 and a.canceled_subscriptions <= a.active_subscriptions):
                    a.cancellation_fee = (a.canceled_subscriptions / a.active_subscriptions) * 100
                else:
                    a.cancellation_fee = 0.0

        @api.depends('arpu', 'active_subscriptions')
        def _ver_arpu(self):
            for a in self:
                if (a.active_subscriptions != 0):
                    a.arpu = a.income_generated / a.active_subscriptions 
                else:
                    a.arpu=0.0