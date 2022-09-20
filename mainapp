from datetime import datetime, date, timedelta
import dash
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from dash import Dash, html, dcc, Input, Output, dash_table
import colorlover
from collections import OrderedDict

# LOAD SEVERAL　FILE DATA
df_new = pd.read_csv('data/NEWCUSTOMER.csv', encoding='cp950')
df_new_trans = pd.melt(df_new, id_vars=['Product Type'], value_vars=list(df_new.columns[1:]), var_name='Year-Month',
                       value_name='COUNT(MDN)')

df_new_trans2 = df_new_trans[df_new_trans['Product Type'] != '總體'].replace('-', 0)
df_new_trans2['COUNT(MDN)'] = df_new_trans2['COUNT(MDN)'].astype(int)

dfAPPMAU = pd.read_csv('data/APPMAU.csv', encoding='cp950')
dfAPPMAU.rename(columns = {'YearMonth':'Year_Month', 'MAU':'活躍用戶數', 'EXIST':'既有用戶', 'NEWCUSTOMER':'首登用戶'}, inplace = True)
dfMINMAX = pd.read_csv('data/APPDAU.csv', encoding='cp950')
dfAPPDAUP = pd.read_csv('data/APPDAUP.csv', encoding='cp950')
dfAPPDAILYNEWUSER = pd.read_csv('data/DAILYNEWUSER.csv', encoding='cp950')
dfAPPACCUMULATION = pd.read_csv('data/ACCUMULATION.csv', encoding='cp950')

# PRE-WORK DATE BY PYTHON AND DATABASE
yesterday = datetime.now() - timedelta(1)
firstdate = yesterday.replace(day=1)

#STYLING DATA TABLE
def highlight_max_row(df):
    df_numeric_columns = dfAPPACCUMULATION.select_dtypes('number').drop(['id'], axis=1)
    return [
        {
            'if': {
                'filter_query': '{{id}} = {}'.format(i),
                'column_id': col
            },
            'background': '#00FFFF',
            'color': 'black'
        }
        # idxmax(axis=1) finds the max indices of each row
        for (i, col) in enumerate(
            df_numeric_columns.idxmax(axis=1)
        )
    ]
dfAPPACCUMULATION['id'] = dfAPPACCUMULATION.index

#STYLING TABS

tabs_styles = {
    'height': '45px',
    'color':'black'
}
tab_style = {
    'borderBottom': '1px solid #d6d6d6',
    'padding': '8px',
    'background-color': '#F0F8FF',
}
tab_selected_style = {
    'borderTop': '1px solid #d6d6d6',
    'borderBottom': '1px solid #d6d6d6',
    'backgroundColor': '#00FFFF',
    'font-size': '20px',
    'fontWeight': 'bold',
    'padding': '8px',
}

#顏色區間
def discrete_background_color_bins(dfAPPDAUP, n_bins=8, columns='all'):

 bounds = [i * (1.0 / n_bins) for i in range(n_bins+1)]
 if columns == 'all':
     if 'id' in dfAPPDAUP:
         df_numeric_columns = dfAPPDAUP.select_dtypes('number').drop(['id'], axis=1)
     else:
         df_numeric_columns = dfAPPDAUP.select_dtypes('number')
 else:
     df_numeric_columns = dfAPPDAUP[columns]
 df_max = df_numeric_columns.max().max()
 df_min = df_numeric_columns.min().min()
 ranges = [
     ((df_max - df_min) * i) + df_min
     for i in bounds
 ]
 styles = []
 legend = []
 for i in range(1, len(bounds)):
     min_bound = ranges[i - 1]
     max_bound = ranges[i]
     backgroundColor = colorlover.scales[str(n_bins)]['seq']['Blues'][i - 1]
     color = 'black'

     for column in df_numeric_columns:
         styles.append({
             'if': {
                 'filter_query': (
                     '{{{column}}} >= {min_bound}' +
                     (' && {{{column}}} < {max_bound}' if (i < len(bounds) - 1) else '')
                 ).format(column=column, min_bound=min_bound, max_bound=max_bound),
                 'column_id': column
             },
             'backgroundColor': backgroundColor,
             'color': color
         })
     legend.append(
         html.Div(style={'display': 'inline-block', 'width': '60px'}, children=[
             html.Div(
                 style={
                     'backgroundColor': backgroundColor,
                     'borderLeft': '1px rgb(50, 50, 50) solid',
                     'height': '10px'
                 }
             ),
             html.Small(round(min_bound, 2), style={'paddingLeft': '2px'})
         ])
     )

 return (styles, html.Div(legend, style={'padding': '5px 0 5px 0'}))

(styles, legend) = discrete_background_color_bins(dfAPPDAUP)


# BUILD APPLICATION
app = Dash(__name__)
server = app.server
# colors = {'text': '#228B22'}  # 'background': '#111111',

# APPLICATION LAYOUT
app.layout = html.Div([
    # FIRST ROW
    html.Div([
        html.Div(
            [html.B(html.H1('Dashboard', style={'textAlign': 'center', 'color': '#7FFF00'}))]),
            ]),
    # SECOND ROW
    html.Div([
        html.Div(
        [html.B(html.H3(
            firstdate.strftime("%Y-%m-%d") + ' ~ ' + str(datetime.strftime(yesterday, '%Y-%m-%d')) +
            ' Accumulate MAU：' + format((dfAPPMAU['活躍用戶數'].iloc[-1]),",")), style={'textAlign': 'center', 'color': 'white'})]),
            ]),
    # THIRD ROW
    html.Div([
        html.Div([
            dcc.Tabs(id = "tabs-styled-with-inline", value = 'tab-1', children = [
                dcc.Tab(label = 'Daily Accumulate User', value = 'tab-1',style = tab_style, selected_style = tab_selected_style),
                dcc.Tab(label = 'Daily New User', value = 'tab-2',style = tab_style, selected_style = tab_selected_style),
                dcc.Tab(label = 'Daily Login User', value = 'tab-3',style = tab_style, selected_style = tab_selected_style),
            ],style = tabs_styles),
            html.Div(id = 'tabs-content-datatable'),
        ], className = "create_container twelve columns", ),
        ], className = "row flex-display"),

    # FOURTH ROW
    html.Div([
        html.Div([
                html.B(html.H4('MAU Composition (MAU = Exist User + New User)', style={'textAlign': 'left', 'color': 'white'})),
                dash_table.DataTable(
                    data=dfAPPMAU.to_dict('records'),
                    columns=[{'id':'Year_Month','name': 'Year_Month','type': 'text'},
                                {'id':'活躍用戶數','name': 'MAU','type': 'numeric'},
                                {'id':'既有用戶','name': 'Exist User','type': 'numeric'},
                                {'id':'首登用戶','name': 'New User','type': 'numeric'}],
                    sort_action='native',
                    style_cell={'textAlign': 'center'},
                    style_data_conditional=[{
                                # 'if': {'row_index': 'odd'},
                                # 'backgroundColor': 'rgb(220, 220, 220)',}],
                                 'backgroundColor': 'rgb(50, 50, 50)',}],
                    style_table={'height': 400},
                    style_header={
                            'backgroundColor': 'rgb(30, 30, 30)',
                            'color': 'white'},
                    style_data={
                            'backgroundColor': 'rgb(50, 50, 50)',
                            'color': 'white'},
                )
            ], className="create_container six columns"),

        html.Div([
                html.B(html.H4('Monthly New User by Product', style={'textAlign': 'left', 'color': 'white'})),
                dcc.Dropdown(id='PTDD', multi=True,
                             options=[{'label': x, 'value': x} for x in sorted(df_new_trans2['Product Type'].unique())],
                             value=['A', 'B', 'C', 'D'], placeholder='Select A Product Type...',style={'color': 'black','backgroundColor': 'rgb(50, 50, 50)'}),
                dcc.Graph(id='example-graph')
            ], className="create_container six columns"),
    ], className="row flex-display"),

    # FIFTH ROW
    html.Div([
        html.Div([
                html.B(html.H4('比較不同區間DAU', style={'textAlign': 'left', 'color': 'white'})),

                html.Div([
                    html.Div([
                html.P('第一區間', style={'color': 'white'}),
                dcc.DatePickerRange(
                    id='date-picker-range',
                    min_date_allowed=date(2022, 1, 1),
                    max_date_allowed=date.today() - timedelta(days=1),
                    start_date_placeholder_text="開始日期",
                    end_date_placeholder_text="結束日期",

                ), ]),
                    html.Div([
                html.P('第二區間', style={'color': 'white'}),
                dcc.DatePickerRange(
                    id='date-picker-range2',
                    min_date_allowed=date(2022, 1, 1),
                    max_date_allowed=date.today() - timedelta(days=1),
                    start_date_placeholder_text="開始日期",
                    end_date_placeholder_text="結束日期",

                ), ]),

                ],className = 'row flex-display'),
             
                dcc.Graph(id='MAU-chart', figure={})
        ], className="create_container six columns"),

        html.Div([
            html.B(html.H4('各產品別留存率', style={'textAlign': 'left', 'color': 'white'})),
            dcc.RadioItems(options=[
                               {'label': 'A', 'value': 'A'},
                               {'label': 'B', 'value': 'B'},
                               {'label': 'C', 'value': 'C'},
                               {'label': 'D', 'value': 'D'},
                                                                ],id='radio-item',value='4G', inline=True),
                                                                
            dcc.Graph(id='Retention-chart', figure={}),
            dcc.Slider(0, 20, 1, value=0, id='my-slider',
                       marks={
                           0: '首月N',
                           1: 'N+1',
                           2: 'N+2',
                           3: 'N+3',
                           4: 'N+4',
                           5: 'N+5',
                           6: 'N+6',
                           7: 'N+7',
                           8: 'N+8',
                           9: 'N+9',
                           10: 'N+10',
                           11: 'N+11',
                           12: 'N+12',
                           13: 'N+13',
                           14: 'N+14',
                           15: 'N+15',
                           16: 'N+16',
                           17: 'N+17',
                           18: 'N+18',
                           19: 'N+19',
                           20: 'N+20',
                       },
                       ),
        ], className="create_container six columns"),
    ], className="row flex-display"),

])

# INTERACTIVE CALLBACK FOR FIGURE
@app.callback(
    Output('Retention-chart', 'figure'),
    #[Input('radio-item', 'value'),
     Input('my-slider', 'value')
)
def update_graph(myslider):
        myslider = myslider+2
        filtered_df = dfRetention_4G.iloc[:, :myslider]
        f = pd.melt(filtered_df, id_vars=['YearMonth'], value_vars=list(filtered_df.columns[1:]), var_name='Month',
                    value_name='Percentage')
        f = f.dropna()
        f['Percentage'] = f['Percentage'].str.rstrip('%').astype('float')
        fig4 = px.line(f, x="Month", y="Percentage", color='YearMonth')
        fig4.update_yaxes(autorange=True)
        return fig4

# INTERACTIVE CALLBACK FOR FIGURE
@app.callback(
    Output('example-graph', 'figure'),
    Input('PTDD', 'value')
)
def update_graph(product_type):
    temp = df_new_trans2[df_new_trans2['Product Type'].isin(product_type)]
    fig = px.bar(temp, x="Year-Month", y="COUNT(MDN)", color="Product Type", text_auto=True)
    fig.update_yaxes(autorange=True)
    return fig

# INTERACTIVE CALLBACK FOR FIGURE
@app.callback(
    Output('MAU-chart', 'figure'),
    [
        Input('date-picker-range', 'start_date'),
        Input('date-picker-range', 'end_date'),
        Input('date-picker-range2', 'start_date'),
        Input('date-picker-range2', 'end_date'),
    ]
)

# DEFINE FUNCTION FOR CALLBACK
def update_charts(start_date, end_date, start_date2, end_date2):
    if not start_date or not end_date:
        raise dash.exceptions.PreventUpdate
    if not start_date2 or not end_date2:
        raise dash.exceptions.PreventUpdate
    else:

        target = ((dfMINMAX.DATE >= start_date) & (dfMINMAX.DATE <= end_date))
        filtered_data = dfMINMAX.loc[target, :]
        filtered_data

        target2 = ((dfMINMAX.DATE >= start_date2) & (dfMINMAX.DATE <= end_date2))
        filtered_data2 = dfMINMAX.loc[target2, :]
        filtered_data2

        fig3 = go.Figure()
        fig3.add_trace(go.Scatter(x=filtered_data['DATE'], y=filtered_data['DAU'],
                                  mode='lines+markers',
                                  name='第一區間'))
        fig3.add_trace(go.Scatter(x=filtered_data2['DATE'], y=filtered_data2['DAU'],
                                  mode='lines+markers',
                                  name='第二區間'))
        # fig3還要加上日期

        return fig3

@app.callback(Output('tabs-content-datatable', 'children'),
              Input('tabs-styled-with-inline', 'value'))
def render_content(tab):
    if tab == 'tab-1':
        return html.Div([
            html.B(html.H4('Daily Accumulate User', style={'textAlign': 'left', 'color': 'white'})),
            dash_table.DataTable(
                data=dfAPPACCUMULATION.to_dict('records'),
                columns=[{"name": i, "id": i} for i in dfAPPACCUMULATION.columns if i != 'id'],
                sort_action='native',
                style_cell={'textAlign': 'left'},
                #style_data_conditional=[{
                #    'backgroundColor': 'rgb(50, 50, 50)', }],
                style_data_conditional = highlight_max_row(dfAPPACCUMULATION),
                style_table={'height': 300},
                style_header={
                    'backgroundColor': 'rgb(30, 30, 30)',
                    'color': 'white'},
                style_data={
                    'backgroundColor': 'rgb(45, 45, 45)',
                    'color': 'white'},
            )
        ])
    elif tab == 'tab-2':
        return html.Div([
            html.B(html.H4('Daily New User', style={'textAlign': 'left', 'color': 'white'})),
            dash_table.DataTable(
                data=dfAPPDAILYNEWUSER.to_dict('records'),
                columns=[{"name": i, "id": i} for i in dfAPPDAILYNEWUSER.columns],
                sort_action='native',
                style_cell={'textAlign': 'left'},
                #style_data_conditional=[{
                 #   'backgroundColor': 'rgb(30, 30, 30)'}],
                style_table={'height': 300},
                style_header={
                    'backgroundColor': 'rgb(30, 30, 30)',
                    'color': 'white'},
                style_data={
                    'backgroundColor': 'rgb(40, 40, 40)',
                    'color': 'white' },
            )
        ])
    elif tab == 'tab-3':
        return html.Div([
            html.Div(legend, style={'float': 'right'}),
            html.B(html.H4('Daily Login User', style={'textAlign': 'left', 'color': 'white'})),
            dash_table.DataTable(
                data=dfAPPDAUP.to_dict('records'),
                columns=[{"name": i, "id": i} for i in dfAPPDAUP.columns],
                sort_action='native',
                style_cell={'textAlign': 'left'},
                style_data_conditional=styles,
                style_table={'height': 300},
                style_header={
                    'backgroundColor': 'rgb(30, 30, 30)',
                    'color': 'white'},
                style_data={
                    'backgroundColor': 'rgb(40, 40, 40)',
                    'color': 'white'},
            )

        ])

if __name__ == '__main__':
    app.run_server(debug=True, use_reloader=False)
